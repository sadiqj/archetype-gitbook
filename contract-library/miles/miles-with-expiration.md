---
description: >-
  This contract archetype manages the expiration date of a mile. An expired mile
  cannot be consumed and may be cleared out by the administrator.
---

# With expiration

A storage optimisation has been done directly in the design of the mile model: a mile is described with an amount field which enables to aggregate miles with same expiration dates. For example, 10 miles with the same expiration date are merged into one mile with the amount value set to 10.

This optimisation comes with a cost of algorithmic complexity in the consume transaction.

{% code-tabs %}
{% code-tabs-item title="miles\_with\_expiration.arl" %}
```ocaml
archetype miles_with_expiration

variable[%transferable] admin role

(* id is a string because it is generated off-chain *)
asset mile identified by id sorted by expiration = {
   id         : string;
   amount     : uint;
   expiration : date
} with {
  m1 : amount > 0
}

(* a partition ensures there is no direct access to mile collection *)
asset owner as role identified by addr = {
  addr  : address;
  miles : mile partition (* injective (owner x mile) *)
}

action add (ow : address) (newmile : mile) = {
   called by admin

   require {
     c1 : newmile.amount > 0
   }

   effect {
     owner.addifnotexist { addr = ow; mile = [] };
     (* action must fail if mile already exists *)
     (owner.get ow).miles.add newmile
   }
}

action consume (a : address) (quantity : uint) = {

  called by admin

  verification {
    invariant loop = {
         (* remainder has a decreasing upper bound *)
         i1 : 0 <= remainder <= to_iter.sum(amount);
         (* removed miles *)
         i2 : (subset (removed miles) (o.miles.select(expiration >= now)));
         (* right amount consumed *)
         i3 : before miles.sum(amount) = miles.sum(amount) + nbmiles - remainder;
         i4 : is_empty added.mile
    }

    specification {
         (* the globla quantity of miles is decreased by nbmiles *)
         p1 : mile.sum(quantity) = before miles.sum(quantity) - nbmiles;
         (* all removed miles are valid miles *)
         p2 : forall m : removed miles, m.expiration >= now;
         (* no added mile *)
         p3 : is_empty added.mile
    }
  }

  effect {
     let ow = owner.get a in
     let by_expiration = ow.miles.select(mile.expiration > now) in
     require (by_expiration.sum(quantity) >= amount);
     let remainder = quantitiy in
     loop : for (m in by_expiration) (
       if remainder > 0
       then (
         if m.amount > remainder
         then (
           remainder := 0;  (* this should be after instruction below
                               this is to demonstrate verification
                               capacity to not verify contract in this
                               state *)
           m.amount  -= remainder
         )
         else if m.amount = remainder
         then (
           remainder := 0;
           o.miles.remove m
         ) else (
           remainder -= m.amount;
           o.miles.remove m
         )
       );
     assert (remainder = 0))
  }
}

(* this ensures that any mile was added with the ‘add’ action *)
specification {
   g1 : anyaction may be performed only by role admin;
   g2 : (remove mile) may be performed only by action (consume or clear_expired);
   g3 : not ((add mile) may be performed only by action consume)
}

action clear_expired = {
  called by admin

  specification {
    s3 : forall m : removed mile, m.expiration < now
  }

  effect {
    for (o in owner) (
      o.miles.removeif (expiration < now)
    )
  }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="miles\_with\_expiration.mlw" %}
```ocaml
module Miles

  use int.Int
  use ref.Ref
  use option.Option
  use list.List
  use list.Append

  use archetype.Types
  use archetype.Contract
  use archetype.Map
  use archetype.AssetCollection
  use archetype.Trace

(* TRACES *)

  type asset =
  | Mile
  | Owner

  type entry =
  | Add
  | Consume
  | ClearExpired

  type field =
  | Amount
  | Expiration
  | Miles

  clone archetype.Trace as Tr with type asset = asset,
                                   type entry = entry,
                                   type field = field

  (* for storage invariant initialisation *)
  val function empty_mile_amounts     : map int
  val function empty_mile_expirations : map date
  val function empty_owner_miles      : map acol
  axiom def_mile_amounts : forall m : key. empty_mile_amounts m > 0

  type storage = {
    mutable admin            : role;
    mutable miles            : acol;
    mutable mile_amounts     : map int;
    mutable mile_expirations : map date;
    mutable owners           : acol;
    mutable owner_miles      : map acol;
    (* contract *)
    mutable ops              : transfers;
    (* diff sets *)
    mutable miles_added      : acol;
    mutable miles_removed    : acol;
    mutable owner_added      : acol;
    mutable owner_removed    : acol;
    (* traces *)
    mutable tr               : Tr.log;
    mutable ename            : option entry;
  } invariant { 
    forall m : key. get mile_amounts m > 0
  } by {
    admin            = 0;
    miles            = Nil;
    mile_amounts     = empty_mile_amounts;
    mile_expirations = empty_mile_expirations;
    owners           = Nil;
    owner_miles      = empty_owner_miles;
    ops              = Nil;
    miles_added      = Nil;
    miles_removed    = Nil;
    owner_added      = Nil;
    owner_removed    = Nil;
    tr               = Nil;
    ename            = None;
  }

  (* getters *)
  let get_mile       (s : storage) (k : key) : key
  raises { NotFound }
  ensures { mem result s.miles }
  = if not (mem k s.miles) then raise NotFound else k

  let function get_amount     (s : storage) (k : key) : int = get s.mile_amounts k

  let function get_expiration (s : storage) (k : key) : date = get s.mile_expirations k

  let get_owner (s : storage) (o : key) : key
  raises { NotFound }
  ensures { mem result s.owners }
  = if not (mem o s.owners) then raise NotFound else o

  (* Owner.miles is a partition of s.miles *)
  axiom subset_miles : forall s : storage, o : key.
    subset (get s.owner_miles o) s.miles

  let function get_miles (s : storage) (o : key) : acol
  ensures { subset result s.miles }
  = get s.owner_miles o


  clone archetype.Sum as Amounts with type     storage = storage,
                                      val      f       = get_amount

  let remove_miles (o : key) (m : key) (s:storage) : unit
  raises { NotFound }
  requires { mem m s.miles }
  requires { mem o s.owners }
  ensures  { forall x:key. mem x s.miles <-> (mem x (old s).miles /\ x <> m) }
  ensures  { forall x:key. mem x s.miles_removed <-> (mem x (old s).miles_removed \/ x = m) }
  ensures  { forall x:key. mem x (get_miles s o) <-> (mem x (get_miles (old s) o) /\ x <> m) }
  ensures  { Amounts.sum (old s) (old s).miles = Amounts.sum s s.miles + get_amount (old s) m }
  =
    s.miles <- remove m s.miles;
    s.miles_removed <- add m s.miles_removed;
    let miles = get_miles s o in
    let new_owner_miles = remove m miles in
    s.owner_miles <- set s.owner_miles o new_owner_miles

  let add_miles (o : key) (m : key) (a : int) (e : date) (s : storage) : unit
  raises { NotFound }
  requires { mem o s.owners }
  requires { not (mem m s.miles) }
  requires { a > 0 }
  ensures  { forall x:key. mem x s.miles <-> (mem x (old s).miles \/ x = m) }
  ensures  { forall x:key. mem x s.miles_added <-> (mem x (old s).miles_added \/ x = m) }
  ensures  { Amounts.sum (old s) (old s).miles + a = Amounts.sum s s.miles }
  =
    s.miles <- add m s.miles;
    s.miles_added <- add m s.miles_added;
    s.mile_amounts <- set s.mile_amounts m a;
    s.mile_expirations <- set s.mile_expirations m e;
    let miles = get_miles s o in
    let new_owner_miles = add m miles in
    s.owner_miles <- set s.owner_miles o new_owner_miles

  let set_amount (m : key) (a : int) (s : storage) : unit
  requires { mem m s.miles }
  requires { a > 0 }
  ensures  { Amounts.sum (old s) (old s).miles = Amounts.sum s s.miles - a + get_amount (old s) m }
  ensures { (old s).miles = s.miles }
  = s.mile_amounts <- set s.mile_amounts m a

  let update_owner (o : key) (new_miles : acol) (s : storage) : unit
  raises { NotFound }
  =
  if not (mem o s.owners) then raise NotFound;
  s.owner_miles <- set s.owner_miles o new_miles

  let addifnotexist (o : key) (new_miles : acol) (s : storage) : unit
  raises { NotFound }
  ensures { mem o s.owners }
  =
  if not (mem o s.owners)
  then (
    s.owners <- add o s.owners;
    update_owner o new_miles s
  )

  (* Actions  *****************************************************************)

  let add (e : env) (s : storage) (ow : key) (m : key) (a : int) (ex : date) : transfers
  raises { InvalidCaller, InvalidCondition, NotFound }
  requires { not (mem m s.miles) }
  =
    if caller e <> s.admin then raise InvalidCaller;
    if not (a > 0) then raise InvalidCondition;
    addifnotexist ow Nil s;
    add_miles ow m a ex s;
    Contract.empty

  (*let function test_consume (e : env) (s : storage) (a : key) : bool = get_expiration s a > now e
  meta "rewrite_def" function test_consume
  axiom tauto_test_consume: forall e : env, s : storage, a : key.
    now e < get_expiration s a <-> test_consume e s a = true

  clone archetype.Filter as FConsume with type storage = storage,
                                          val  test    = test_consume*)

  let rec function filter_consume (e : env) (s : storage) (c : acol) : acol
  variant { c }
  ensures { forall a : key. mem a result -> get_expiration s a > now e }
  ensures { subset result c }
  =
  match c with
  | Nil -> Nil
  | Cons a tl ->
    if get_expiration s a > now e
    then Cons a (filter_consume e s tl)
    else filter_consume e s tl
  end

  let consume (e : env) (s : storage) (owner : address) (nbmiles : int) : transfers
  raises { NotFound, InvalidCaller, InvalidCondition }
  requires { is_empty s.miles_removed }
  requires { is_empty s.miles_added }
  (* forall m : removed miles, m.expiration > now *)
  ensures { forall m : key.
     (*mem m (diff (old s).miles s.miles) ->  get_expiration s m > (now e)*)
     mem m s.miles_removed ->  get_expiration s m > (now e)
  }
  (* mile.sum(quantity) = before miles.sum(quantity) - nbmiles *)
  ensures { Amounts.sum (old s) (old s).miles = Amounts.sum s s.miles + nbmiles }
  (* is_emtpy add.miles *)
  ensures { is_empty s.miles_added }
  = if caller e <> s.admin then raise InvalidCaller;
    if nbmiles <= 0 then raise InvalidCondition;
    let o = get_owner s owner in
    let miles = get_miles s o in
    let l = filter_consume e s miles in
    (*let l = FConsume.filter e s miles in*)
    if not (nbmiles <= Amounts.sum s l) then raise InvalidCondition;
    let remainder = ref nbmiles in
    try
      for i = 0 to (length l) - 1 do
      (* helps for the membership precondition of set_amount: *)
      invariant { forall k : int. i <= k < length l ->  mem (nth k l) s.miles }
      (* removed miles are in l: *)
      (*invariant { subset (diff (old s).miles s.miles) l }*)
      invariant { subset s.miles_removed l }
      (* remainder bounds: *)
      invariant { 0 <= !remainder <= Amounts.part_sum s l i (length l) }
      (* right amount spent invariant *)
      invariant { Amounts.sum (old s) (old s).miles = Amounts.sum s s.miles + nbmiles - !remainder }
        let m = nth i l in
        if get_amount s m > !remainder
        then
          (set_amount m (get_amount s m - !remainder) s;
           remainder := 0;
           raise Break)
        else if get_amount s m = !remainder
        then
          (remainder := 0;
           remove_miles o m s;
           raise Break)
        else
          (remainder := !remainder - get_amount s m;
           remove_miles o m s)
      done;
    with Break -> assert { !remainder = 0 }; ()
    end;
    assert { !remainder = 0 };
    Contract.empty

(*    let function test_clear_expired (e : env) (s : storage) (a : key) : bool = get_expiration s a < now e

    clone archetype.Filter as FClearexpired with type storage = storage,
                                                 val  test    = test_clear_expired*)

    let rec function filter_clear_expired (e : env) (s : storage) (c : acol) : acol
    variant { c }
    ensures { forall a : key. mem a result -> get_expiration s a < now e }
    ensures { subset result c }
    =
    match c with
    | Nil -> Nil
    | Cons a tl ->
      if get_expiration s a < now e
      then Cons a (filter_clear_expired e s tl)
      else filter_clear_expired e s tl
    end

    let clear_expired (e : env) (s : storage) : transfers
    raises { NotFound, InvalidCaller }
    requires { is_empty s.miles_added }
    requires { is_empty s.miles_removed }
    ensures { forall m : key. mem m s.miles_removed -> get_expiration s m < now e }
    = if caller e <> s.admin then raise InvalidCaller;
      for i = 0 to (length s.owners)-1 do
      invariant { forall m : key. mem m s.miles_removed -> get_expiration s m < now e }
        let o = nth i s.owners in
        let miles = get_miles s o in
        let l = filter_clear_expired e s miles in
        label Internal_loop in
        for j = 0 to (length l) - 1 do
        invariant { forall k : int. j <= k < length l -> mem (nth k l) s.miles }
        invariant { forall m : key. mem m s.miles_removed <-> 
          (mem m (s at Internal_loop).miles_removed \/ (exists k : int. 0 <= k < j /\ m = nth k l)) }
          let m = nth j l in
          remove_miles o m s;
        done;
        
      done;
      Contract.empty

  let all_entries (e : env) (s : storage)
                  (* add *)
                  (ow : key) (m : key) (a : int) (ex : date)
                  (* consume *)
                  (owner : address) (nbmiles : int)
                  (* clear expired *)
  raises { NotFound, InvalidCaller, InvalidCondition }   
  requires { s.tr = Nil }
  requires { s.ops = Nil }             
  (* any action is performed only by rome admin *)
  ensures { forall ac : Tr.action, a : asset. Tr.performed_only_by_role e s.tr (Cons ac Nil) (Cons a Nil) (Cons s.admin Nil) }
  (* remove miles is performed only by action (consome or clear_expired)*)
  ensures { Tr.performed_only_by_action e s.tr (Cons Tr.Rm Nil) (Cons Mile Nil) (Cons Consume (Cons ClearExpired Nil)) }
  (* add is not permformed by consume *)
  =
  match s.ename with
   | Some Add -> 
    if caller e <> s.admin then raise InvalidCaller;
    if not (a > 0) then raise InvalidCondition;
    s.tr <- s.tr ++ Tr.mk_trace Add Owner Tr.Add;
    s.tr <- s.tr ++ Tr.mk_trace Add Mile Tr.Add;
    Contract.empty
   | Some Consume -> 
    if caller e <> s.admin then raise InvalidCaller;
    if nbmiles <= 0 then raise InvalidCondition;
    let o = get_owner s owner in
    let miles = get_miles s o in
    let l = filter_consume e s miles in
    if not (nbmiles <= Amounts.sum s l) then raise InvalidCondition;
    let remainder = ref nbmiles in
    try
      for i = 0 to (length l) - 1 do
      invariant { forall ac : Tr.action, a : asset. Tr.performed_only_by_role e s.tr (Cons ac Nil) (Cons a Nil) (Cons s.admin Nil) }
      invariant { Tr.performed_only_by_action e s.tr (Cons Tr.Rm Nil) (Cons Mile Nil) (Cons Consume (Cons ClearExpired Nil)) }
        let m = nth i l in
        if get_amount s m > !remainder
        then
          (s.tr <- s.tr ++ Tr.mk_trace Consume Mile (Tr.Update Amount);
           remainder := 0;
           raise Break)
        else if get_amount s m = !remainder
        then
          (remainder := 0;
           s.tr <- s.tr ++ Tr.mk_trace Consume Mile Tr.Rm;
           raise Break)
        else
          (remainder := !remainder - get_amount s m;
           s.tr <- s.tr ++ Tr.mk_trace Consume Mile Tr.Rm)
      done;
    with Break -> ()
    end;
    Contract.empty
   | Some ClearExpired ->
      if caller e <> s.admin then raise InvalidCaller;
      for i = 0 to (length s.owners)-1 do
      invariant { forall ac : Tr.action, a : asset. Tr.performed_only_by_role e s.tr (Cons ac Nil) (Cons a Nil) (Cons s.admin Nil) }
      invariant { Tr.performed_only_by_action e s.tr (Cons Tr.Rm Nil) (Cons Mile Nil) (Cons Consume (Cons ClearExpired Nil)) }
        let o = nth i s.owners in
        let miles = get_miles s o in
        let l = filter_clear_expired e s miles in
        for j = 0 to (length l) - 1 do
        invariant { forall ac : Tr.action, a : asset. Tr.performed_only_by_role e s.tr (Cons ac Nil) (Cons a Nil) (Cons s.admin Nil) }
        invariant { Tr.performed_only_by_action e s.tr (Cons Tr.Rm Nil) (Cons Mile Nil) (Cons Consume (Cons ClearExpired Nil)) }
          let m = nth j l in
          s.tr <- s.tr ++ Tr.mk_trace Consume Mile Tr.Rm
        done;
        
      done;
      Contract.empty
   | None -> Contract.empty
  end
end

```
{% endcode-tabs-item %}

{% code-tabs-item title="miles\_with\_expiration.md" %}
```bash
# Miles with Expiration
> Genrated with [Archetype](https://archetype-lang.org/) v0.1 (2019-04-09)
## Roles

### Admin

|                |   |
|----------------|---|---|
|Address         |   |
|Extension       |   |

## Assets
### Mile

| Field | Desc | Type | Attribute |
|--|--|--|--|
| id |  | string | __key__, order
| quantity |  | integer |
| expiration |  | date | 
### Owner

| Field | Desc | Type | Attribute |
|--|--|--|--|
| addr |  | string | __key__, order
| miles |  | partition | 

## Actions

### add
`called by` admin
| Name | Desc | Type |  
|--|--|--|
|ow||string
|newmile||mile asset

### consume
`called by` admin
| Name | Desc | Type |  
|--|--|--|
|ow||string
|val||integer
#### Formal Specification
##### p1
`let o = owner.get owner in o.miles.sum(quantity) = before o.miles.sum(quantity) - val`
##### p2
`forall m : removed miles, m.expiration >= now`
##### p3
`is_empty (added miles)`
### clear
`called by` admin

## Formal properties

##### m1
`forall m : mile, m.amount > 0`
##### s1
`anyaction  may be performed only by role admin`
##### s2
`(remove mile) may be performed only by tx consume`
##### s3
`transfers_by_tx  anytx = 0`
```
{% endcode-tabs-item %}
{% endcode-tabs %}



