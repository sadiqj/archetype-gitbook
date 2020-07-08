# Reference

An archetype contract is composed of declarations, actions \(aka entry points\) and optionally specification for formal verification.

## Declarations

* `constant` and `variables`  declare global variables. A constant value cannot be modified in the contract's actions.

```ocaml
constant rate : rational = 0.2
```

```ocaml
variable price : tez = 10tz
```

* `states`  declares contract's possible states. It is then possible to use _transitions_ to change the contract states \(see below\)

```ocaml
states =
| Created initial
| Confirmed
| Canceled
| Success
| Fail
```

 `initial`  is used to declare the initial state value of the contract. 

* `entry` declares an entry point of the contract. An action has the following sections:
  * `specification` \(optional\) to provide the _post conditions_ the action is supposed to have
  * `accept transfer` \(optional\) to specify that transfer of tez is accepted 
  * `called by` \(optional\) to declare which role may call this action
  * `require` \(optional\) to list the necessary conditions for the action to be executed
  * `failif` \(optional\)to list the conditions which prevent from execution 
  * `effect` is the code to execute by the action

```css
entry an_entry_1 (arg1 : string, arg2 : int) {
  specification {
    // see 'specification' section below
  }
  called by a_role
  require {
    r1 : arg2 > 0 
  }
  failif {
    f1 : arg2 > 10
  }
  effect {
    // ...
  }
}
```

An entry with just an effect section may omit the effect section syntax:

```css
variable val : int = 0

entry add(v : int) {
  val += v;
}
```

* `transition` declares an entry point of the contract that changes the state of the contract. A transition has the same sections as an action \(except `effect`, see above\) plus:
  * `from` to specify the states the transition starts from
  * `to` to specify the states after the transition 
  * `when` \(optional\) to specify the transition condition
  * `with effect` to specify the effect of the transition

```c
states =
| Created initial
| Confirmed
| Canceled
| Success
| Fail

transition confirm () {
  from Created
  to Confirmed
  when { transferred > 10tz }
  with effect {
    transfer 1tz to @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg
  }
}
```

It is possible to specify several `to ... when .... with effect ...` sections in a transition. It is possible to specify a list of original states for the transition to start from:

```c
transition to_fail () {
  from Created or Confirmed
  to Fail
  when { ... }
}
```

## Builtin types

* `bool` : boolean values
* `int` : integer values 
* `nat` : positive integers
* `rational` : floating value that can be expressed as the quotient or fraction of two integers 
* `address` : account address
* `role` : an address that can be used in action's called by section
* `date` : date values
* `duration` : duration values \(in second, minute, hour, day, week\)
* `string` : string of characters
* `tez` : Tezos currency
* `bytes` : bytes sequence
* `key` : key value
* `signature` : signature value
* `key_hash` : key hash value
* `chain_id` : chain id value

## Composite types

* `*` declares a tuple made of other types.

```c
constant pair : int * string = (1, "astr")
```

* `option` declares an option of type.

```csharp
constant value : option<int> = none
```

* `list` declares a list of any type \(builtin or composed\)

```csharp
variable vals : list<string> = []
```

* `map` declares a map from a non-composite builtin type to any type.

```csharp
variable assoc : map<int, string> = []
```

* `set` declares a set of non-composite builtin type values.

```csharp
variable poll : set<address> = []
```

* `record` declares a record structure.

```css
record r = {
   s : string;
   i : int;
}
```

* `asset`  declares a collection of assets and the data an asset is composed of. For example, the following declares a collection of real estates described by an address, a location, and an owner:

```css
asset real_estate identified by addr {
  addr      : string;
  xlocation : string;
  ylocation : string;
  owner     : address;
}
```

* `enum`  declares an enumeration value. It is read with a `match ... with` command \(see below\). 

```ocaml
enum color =
| White 
| Red
| Green
| Blue
```

It is then possible to declare a variable of type _color_:

```css
variable c : color = Green
```

* `contract` declares the signature of another existing contract to call in actions.

```css
contract called_contract_sig {
   entry set_value (n : int)
   entry add_value (a : int, b : int)
}
```

It is then possible to declare a contract value of type _called\_contract\_sig_ and set its address:

```c
constant c : called_contract_sig = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm
```

* `aggregate` declares an asset field as an aggregate of another asset.

```css
asset an_asset {
  id     : string;
  a_val  : int;
  col    : aggregate<another_asset>;
}
```

* `partition` declares an asset field as a collection of another asset. The difference with `aggregate` is that a partition ensures at compilation that every _partitioned_ asset \(i.e. element of the partition\) belongs to one and only _partitioning_ asset.

```css
asset partitioning_asset {
  id   : string;
  part : partition<partionned_asset>;
}
```

As a consequence of the partition, a _partitioned_ asset cannot be straightforwardly added or removed to its global collection with `add` and `remove` \(see [Partitions, Reference](archetype-language/subsets-partition.md) section\). 

## Effect

An action's effect is composed of instructions separated by a semi-colon. 

### Declaration

* `var` declares a local variable with its value. While possible, it is not necessary to declare the type a local variable. Local variables' identifier must be unique \(no name capture in Archetype\).

```javascript
var i = 0;
```

### Assignment

* `:=` enables to assign a new value to a global or local variable. 

```javascript
i := 1;
```

* `+=` , `-=` , `*=` and `/=` enable to respectively increment and decrement an integer variable \(local or global\).

```javascript
i *= 2;  // i := i * 2
d += 3w; // d := i + 3w, date d is now previous d plus 3 weeks
```

### Effects on asset collection

* `add` enables to add an asset to an asset collection. It fails if the asset key is already present in the collection.

```javascript
an_asset.add({ id = "RJRUEQDECSTG", asset_col = [] }); // see 'collection' example
```

* `update` enables to update an existing asset; it takes as arguments the id of the asset to update and the list of effects on the asset. It fails if the id is not present in the asset collection.

```javascript
an_asset.update("RJRUEQDECSTG", { a_value += 3 });
```

* `addupdate` is similar to update except that it adds the asset if its identifier is not present in the collection.

```javascript
an_asset.addupdate("RJRUEQDECSTG", { a_value += 3 });
```

* `remove` removes a an asset from its collection. 

```javascript
an_asset.remove("RJRUEQDECSTG");
```

* `removeif` enables removing assets under a condition.

```c
an_asset.removeif(a_val > 10); // "a_val" is an asset field
```

* `clear` clears an asset collection.

```javascript
an_asset.clear();
```

### Control

* `;` sequence

```javascript
res := 1;
res := 2;
res := 3;
```

* `if then` and `if then else` are the conditional branchings instructions. 

```ocaml
if i > 0 then (
  // (* effect when i > 0 *) 
) else (
  // (* effect when i <= 0 *)
);
```

* `match ... with ... end` 

```javascript
archetype effect_control_matchwith

enum t =
  | C1
  | C2
  | C3
  | C4
  | C5

variable res : int = 0

entry exec () {
  specification {
    s0: res = 1;
  }
  effect {
    var x : t = C3;
    match x with
    | C1 | C2 -> res := 0
    | C3 -> res := 1
    | _ -> res := 2
    end
  }
}

```

* `for in do done` iterates over a collection.

```ocaml
 var res = 0;
 for a in an_asset do
   res += a.a_val;
 done;
```

* `iter to do done` iterates over a sequence of integer values \(starting from 1\).

```javascript
var res = 0
iter i to 3 do      // iterates from 1 to 3 included
  res += i;
done;               // res is 6
```

* `transfer` transfers an amount of tez to an address or a contract.

```c
transfer 2tz to @tz1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm;
```

With the `contract` keyword presented above, it is possible to transfer to a contract and call an entry point. In the example below, the entry point `set_value` of contract `c` is called and `2tz` is transferred.

```c
contract contract_called_sig {
   entry set_value (n : int)
   entry add_value (a : int, b : int)
}

constant c : called_contract_sig = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm

entry update_value(n : int) {
  effect {
    transfer 2tz to c call set_value(n);
  }
}
```

* `fail` aborts the execution. It prevents from deploying the contract on the blockchain. As a consequence the storage is left unchanged when executed.

```c
fail("a message");
```

* `require` and `failif` fail if the argument condition is respectively false and true. `require(t)` is  sugar for `if t then fail("")`.

```c
require(val > 0);
failif(val <= 0);
```

## Expressions

### Literals

* `boolean`

```javascript
constant x : bool = true
constant y : bool = false
```

* `integer`

```c
constant i : int = 1
constant j : int = -42
```

* `rational`

```c
constant f : rational = 1.1
constant g : rational = -1.1
constant r : rational = 2 div 6
constant t : rational = -2 div 6
```

* `string`

```c
constant s : string = "hello world"
```

* `tez`

```c
constant ctz  : tez = 1tz
constant cmtz : tez = 1mtz
constant cutz : tez = 1utz
```

* `address` / `role`

```c
constant a : address = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg
```

* `duration`

```c
// duration of 3 weeks 8 days 4 hours 34 minutes 18 seconds
constant d : duration = 3w8d4h34m18s 
```

* `date`

```c
constant date0 : date = 2019-01-01                
constant date1 : date = 2019-01-01T01:02:03       
constant date2 : date = 2019-01-01T01:02:03Z      
constant date3 : date = 2019-01-01T00:00:00+01:00 
constant date4 : date = 2019-01-01T00:00:00-05:30 
```

* `list`

```c
constant mylist : int list = [1; 2; 3]
```

* `option`

```c
constant op1 : int option = none
constant op2 : int option = some(0)
```

* `bytes`

```c
constant bl : bytes = 0x12f12354356a 
```

### Operators

#### Logical

* `and` operator of logical conjunction

```javascript
var bool_bool_and : bool = (true and false);
```

* `or` operator of logical disjunction

```javascript
var bool_bool_or  : bool = (true or false);
```

* `not` operator of logical negation

```javascript
var bool_bool_not : bool = not true;
```

### Operator

* `=` 

```javascript
var int_int_eq   : bool = 1 = 2;
var rat_int_eq   : bool = (1 / 2) = 2;
var int_rat_eq   : bool = 1 = (2 / 3);
var rat_rat_eq   : bool = (1 / 3) = (2 / 3);
var tez_tez_eq   : bool = 1tz = 2tz;
var dur_dur_eq   : bool = 1h = 2h;
var date_date_eq : bool = 2020-01-01 = 2020-12-31;
var bool_bool_eq : bool = true = false;
var addr_addr_eq : bool = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg = @tz1bfVgcJC4ukaQSHUe1EbrUd5SekXeP9CWk;
var str_str_eq   : bool = "a" = "b";
```

* `<>` 

```javascript
var int_int_ne   : bool = 1 <> 2;
var rat_int_ne   : bool = (1 / 2) <> 2;
var int_rat_ne   : bool = 1 <> (2 / 3);
var rat_rat_ne   : bool = (1 / 2) <> (2 / 3);
var tez_tez_ne   : bool = 1tz <> 2tz;
var dur_dur_ne   : bool = 1h <> 2h;
var date_date_ne : bool = 2020-01-01 <> 2020-12-31;
var bool_bool_ne : bool = true <> false;
var addr_addr_ne : bool = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg <> @tz1bfVgcJC4ukaQSHUe1EbrUd5SekXeP9CWk;
var str_str_ne   : bool = "a" <> "b";
```

* `<` 

```javascript
var int_int_lt   : bool = 1 < 2;
var rat_int_lt   : bool = (1 / 2) < 2;
var int_rat_lt   : bool = 1 < (2 / 3);
var rat_rat_lt   : bool = (1 / 2) < (2 / 3);
var tez_tez_lt   : bool = 1tz < 2tz;
var dur_dur_lt   : bool = 1h < 2h;
var date_date_lt : bool = 2020-01-01 < 2020-12-31;
```

* `<=` 

```javascript
var int_int_le   : bool = 1 <= 2;
var rat_int_le   : bool = (1 / 2) <= 2;
var int_rat_le   : bool = 1 <= (2 / 3);
var rat_rat_le   : bool = (1 / 2) <= (2 / 3);
var tez_tez_le   : bool = 1tz <= 2tz;
var dur_dur_le   : bool = 1h <= 2h;
var date_date_le : bool = 2020-01-01 <= 2020-12-31;
```

* `>` 

```javascript
var int_int_gt   : bool = 1 > 2;
var rat_int_gt   : bool = (1 / 2) > 2;
var int_rat_gt   : bool = 1 > (2 / 3);
var rat_rat_gt   : bool = (1 / 2) > (2 / 3);
var tez_tez_gt   : bool = 1tz > 2tz;
var dur_dur_gt   : bool = 1h > 2h;
var date_date_gt : bool = 2020-01-01 > 2020-12-31;
```

* `>=` 

```javascript
var int_int_ge   : bool = 1 >= 2;
var rat_int_ge   : bool = (1 / 2) >= 2;
var int_rat_ge   : bool = 1 >= (2 / 3);
var rat_rat_ge   : bool = (1 / 2) >= (2 / 3);
var tez_tez_ge   : bool = 1tz >= 2tz;
var dur_dir_ge   : bool = 1h >= 2h;
var date_date_ge : bool = 2020-01-01 >= 2020-12-31;
```

### Arithmetic

* `+` 

```javascript
var int_int_plus   : int      = 1 + 2;
var rat_rat_plus   : rational = 1.1 + 1.2;
var int_rat_plus   : rational = 1 + 1.2;
var rat_int_plus   : rational = 1.1 + 2;
var dur_dur_plus   : duration = 1h + 2s;
var date_dur_plus  : date     = 2020-01-01 + 1d;
var dur_date_plus  : date     = 1d + 2020-01-01;
var str_str_plus   : string   = "str1" + "str2"; (* concat *)
var tez_tez_plus   : tez      = 2tz + 1tz;
```

* `-` 

```javascript
var int_int_minus   : int       = 1 - 2;
var rat_rat_minus   : rational  = 1.1 - 1.2;
var int_rat_minus   : rational  = 1 - 1.2;
var rat_int_minus   : rational  = 1.1 - 2;
var date_date_minus : duration  = 2020-01-01 - 2019-12-31; (* absolute value *)
var dur_dur_minus   : duration  = 1h - 2s; (* absolute value *)
var date_dur_minus  : date      = 2020-01-01 - 1d;
var tez_tez_minus   : tez       = 2tz - 1tz;
```

* `*` 

```javascript
var int_int_mult   : int      = 1 * 2;
var rat_rat_mult   : rational = 1.1 * 1.2;
var int_rat_mult   : rational = 1 * 1.2;
var rat_int_mult   : rational = 1.1 * 2;
var int_dur_mult   : duration = 2 * 1h;
var int_tez_mult   : tez      = 1 * 1tz;
var rat_tez_mult   : tez      = 1.1 * 1tz;
```

* `/` 

```javascript
var int_int_div    : rational = 1 / 2;
var rat_rat_div    : rational = 1.1 / 1.2;
var int_rat_div    : rational = 1 / 1.2;
var rat_int_div    : rational = 1.1 / 2;
```

* `%` 

```javascript
var int_int_modulo : int = 1 % 2;
```

### Constant

* `caller` 

```javascript
var v : address = caller; (* SENDER *)
```

* `source` 

```javascript
var v : address = source; (* SOURCE *)
```

* `balance` 

```javascript
var v : tez = balance; (* BALANCE *)
```

* `transferred` 

```javascript
var v : tez = transferred; (* AMOUNT *)
```

* `now` 

```javascript
var v : date = now; (* NOW *)
```

* `state` 

```javascript
archetype sample_state

states =
| First
| Second
| Third


transition mytr () {
  from First
  to Second
  with effect {
    require (state = First)
  }
}
```

### Built-in functions

* `min` 

```javascript
var int_int_min   : int  = min(1, 2);
var rat_int_min   : rat  = min(1 / 2, 1);
var int_rat_min   : rat  = min(2, 1 / 3);
var rat_rat_min   : rat  = min(1 / 2, 1 / 3);
var date_date_min : date = min(2020-01-01, 2020-12-31);
var dur_dur_min   : dur  = min(1h, 1s);
var tez_tez_min   : tez  = min(1tz, 2tz);
```

* `max` 

```javascript
var int_int_max   : int  = max(1, 2);
var rat_int_max   : rat  = max(1 / 2, 1);
var int_rat_max   : rat  = max(2, 1 / 3);
var rat_rat_max   : rat  = max(1 / 2, 1 / 3);
var date_date_max : date = max(2020-01-01, 2020-12-31);
var dur_dur_max   : dur  = max(1h, 1s);
var tez_tez_max   : tez  = max(1tz, 2tz);
```

* `abs` 

```javascript
var int_abs : int = abs(-1);
var rat_abs : rat = abs(-1 / 2);
```

* `concat`

```javascript
var str_concat : string = concat("abc", "def");
var byt_concat : bytes  = concat(0x12, 0xef);
```

* `slice`

```javascript
var str_slice : string = slice("abcdef", 1, 2);
var byt_slice : bytes  = slice(0xabcdef01, 1, 2);
```

* `length`

```javascript
var str_length : int = length("abcdef");
```

* `floor`

```javascript
var pos_floor : int = floor(5 / 3);  // 1
var neg_floor : int = floor(-5 / 3); // -2
```

* `ceil`

```javascript
var pos_ceil : int = ceil(5 / 3);  // 2
var neg_ceil : int = ceil(-5 / 3); // -1
```

* `pack`

```javascript
var v : bytes = pack("archetype")
```

* `unpack`

```javascript
var s : string option = unpack<string>(0x050100000009617263686574797065) // some("archetype")
```

* `entrypoint`

```cpp
variable e : option<entrysig<nat>> = entrypoint(@KT1,"getbalance")
```

### Option

* `issome`
* `isnone`
* `getopt`

```css
effect {
    var t1 : bool = isnone(none);
    var t2 : bool = issome(some(1));
    var i  : int = getopt(some(1)); /* i = 1 */
}
```

### List

* `contains`
* `count`
* `nth`
* `prepend`

```css
effect {
    var l : string list = ["1"; "2"; "3"];
    var t1 = contains(l, "2");
    var t2 = count(l);
    var n  = nth(l);
    var p  = prepend(l,"0");
}
```

### Set

* `set_add`
* `set_remove`
* `set_contains`
* `set_length`

```css
effect {
    var my_set : set<int> = [0; 1 ; 2; 3];
    var new_set2 : set<int> = set_add(my_set, 4);
    var new_set3 : set<int> = set_remove(my_set, 0);
    var set_c    : bool     = set_contains(my_set, 2);
    var c        : int      = set_length(my_set);
}
```

### Map

* `map_put`
* `map_remove`
* `[]`
* `map_getopt`
* `map_contains`
* `map_length`

```css
effect {
    var my_map : map<string, int> = [ ("k0", 3) ;
                                      ("k1", 2) ;
                                      ("k2", 1) ;
                                      ("k3", 0) ];
    var new_map1 : map<string, int> = map_put(my_map, "k4", 4);
    var new_map2 : map<string, int> = map_remove(my_map, "k0");
    var new_map3 : option<int>      = my_map["k0"];
    var new_map4 : int              = map_getopt(my_map, "k0");
    var map_c    : bool             = map_contains(my_map, "k0");
    var map_l    : int              = map_length(my_map);
}
```

### Record



### Asset Collection

* `contains`
* `count`
* `nth`
* `head`
* `tail`
* `select`
* `sort`
* `sum`

```css
archetype expr_method_asset_contains

asset my_asset identified by id {
  id    : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

variable res : bool = false

entry exec () {
  specification {
    s0 : res = true;
  }
  effect {
    var t = my_asset.contains("id0");
    var c = my_asset.count();
    var n = my_asset.nth(1);
    var l1 = my_asset.head(3);
    var l2 = my_asset.tail(2);
    var l3 = my_asset.select(the.id < "id2");
    var l4 = my_asset.sort(value);
    var l5 = my_asset.sort(v1, asc(v2), desc (v3));
    var s = my_asset.sum(value);
  }
}
```

## Verification

### Specification

Here is a full example with all sections of an action specification

```css
archetype contract_with_full_specification

asset myasset {
  id : string;
  val: bool;
}

asset col1 {
  id1 : string;
}

asset col2 {
  id2 : string;
}

entry exec () {
 specification {

    definition mydef {
      x : myasset | forall y in col1, x.id = y.id1
    }

    predicate mypredicate (a : int) {
      forall x in col1, forall y in col2, x.id1 = y.id2
    }

    variable myvar : int = 0

    shadow effect {
      myvar := 3
    }

    assert a1 {
      x = y
      invariant for myloop {
        x = 0;
        y = 0
      }
    }

    postcondition s1 {
      x = y
      invariant for myloop {
        x = 0;
        y = 0
      }
    }
  }

  effect {
    var x : int = 0;
    var y : int = 0;
    for : myloop c in col1 do
      require(true)
    done;
    assert a1
  }
}

```

* `definition`

```css
definition mydef {
  x : myasset | forall y in col1, x.id = y.id1
}
```

* `predicate` 

```css
predicate mypredicate (a : int) {
  forall x in col1, forall y in col2, x.id1 = y.id2
}
```

* `variable` 

```css
variable myvar : int = 0
```

* `shadow effect` 

```css
shadow effect {
  myvar := 3
}
```

* `assert` 

```css
assert a1 {
  x = y
  invariant for myloop {
    x = 0;
    y = 0
  }
}
```

* `invariant` 

```css
invariant for myloop {
  x = 0;
  y = 0
}
```

* `postcondition` 

```css
postcondition s1 {
  x = y
  invariant for myloop {
    x = 0;
    y = 0
  }
}
```

One specification section is available at the top of the contract. But there is neither `shadow effect` nor `postcondition`, which is replaced by `contract invariant` for the last one.

* `contract invariant` 

```css
contract invariant c1 {
  true <> false
}
```

### Expression

All expression in effect are available in formula part.

* `forall` universal quantifier

```css
q1: forall x : int, x = x;
q2: forall x in my_asset, x.value = x.value;
```

* `exists` existential quantifier

```css
q3: exists x : int, x = x;
q4: exists x in my_asset, x.value = x.value;
```

* `->` imply

```css
o1: true -> true;
```

* `<->` equivalence

```css
o2: true <-> true;
```

### Asset expression

* `subsetof` 

```css
archetype expr_formula_asset_method_subset

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

entry exec () {

  specification {
    s: my_asset.subsetof(my_asset);
  }

  effect {
    require (true)
  }
}
```

* `isempty` 

```css
archetype expr_formula_asset_method_isempty

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

entry exec () {

  specification {
    s: my_asset.isempty();
  }

  effect {
    require (true)
  }
}
```

### Extra asset collection

* `before` 

```css
s1: before.my_asset.isempty();
```

* `at` 

```css
s2: at(lbl).my_asset.isempty();
```

* `unmoved` 

```css
s3: unmoved.my_asset.isempty();
```

* `added` 

```css
s4: added.my_asset.isempty();
```

* `removed` 

```css
s5: removed.my_asset.isempty();
```

* `iterated` 

```css
entry exec () {
  specification {
    postcondition p1 {
      true
      invariant for loop {
         iterated.isempty()
      }
    }
  }

  effect {
    var res : int = 0;
    for:loop i in my_asset do
      res += 1;
    done
  }
}
```

* `toiterate` 

```css
action exec () {
  specification {
    postcondition p1 {
      true
      invariant for loop {
         toiterate.isempty() 
      }
    }
  }

  effect {
    var res : int = 0;
    for:loop i in my_asset do
      res += 1;
    done
  }
}
```

### Security predicate

```css
archetype lang_security

constant admin : role = @tz1aazS5ms5cbGkb6FN1wvWmN7yrMTTcr6wB

asset my_asset identified by id {
  id : string;
  value : int;
}

entry exec () {
  effect {
    ()
  }
}

security {
  s00 : only_by_role (anyentry, admin);
  s01 : only_in_entry (anyentry, exec);
  s02 : only_by_role_in_entry (anyentry, admin, exec);
  s03 : not_by_role (anyentry, admin);
  s04 : not_in_entry (anyentry, exec);
  s05 : not_by_role_in_entry (anyentry, admin, exec);
  s06 : transferred_by (anyentry);
  s07 : transferred_to (anyentry);
  s08 : no_storage_fail (anyentry);
}

```

* `only_by_role` 

```css
s00 : only_by_role (anyentry, admin);
```

* `only_in_action` 

```css
s01 : only_in_entry (anyentry, exec);
```

* `only_by_role_in_action` 

```css
s02 : only_by_role_in_entry (anyentry, admin, exec);
```

* `not_by_role` 

```css
s03 : not_by_role (anyentry, admin);
```

* `not_in_action` 

```css
s04 : not_in_entry (anyentry, exec);
```

* `not_by_role_in_action` 

```css
s05 : not_by_role_in_entry (anyentry, admin, exec);
```

* `transferred_by` 

```css
s06 : transferred_by (anyentry);
```

* `transferred_to` 

```css
s07 : transferred_to (anyentry);
```

* `no_storage_fail` 

```css
s08 : no_storage_fail (anyentry);
```

