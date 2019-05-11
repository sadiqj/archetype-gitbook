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
   g2 : (remove mile) may be performed only by tx (consume or clear_expired);
   g3 : not ((add mile) may be performed only by tx consume)
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
{% endcode-tabs %}



