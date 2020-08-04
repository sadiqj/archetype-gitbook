# Reference

An archetype contract is composed of declarations, entry points and optionally specification for formal verification.

## Declarations

* `constant` and `variables`  declare global variables. A constant value cannot be modified in the contract's entry points.

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

`initial` is used to declare the initial state value of the contract.

* `entry` declares an entry point of the contract. An entry has the following sections:
  * `specification` \(optional\) to provide the _post conditions_ the entry is supposed to have
  * `accept transfer` \(optional\) to specify that transfer of tez is accepted 
  * `called by` \(optional\) to declare which role may call this entry
  * `require` \(optional\) to list the necessary conditions for the entry to be executed
  * `failif` \(optional\)to list the conditions which prevent from execution 
  * `effect` is the code to execute by the entry

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

* `transition` declares an entry point of the contract that changes the state of the contract. A transition has the same sections as an entry \(except `effect`, see above\) plus:
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
* `rational` : floating value that can be expressed as the quotient or fraction of two integers 
* `address` : account address
* `role` : an address that can be used in entry's called by section
* `date` : date values
* `duration` : duration values \(in second, minute, hour, day, week\)
* `string` : string of characters
* `tez` : Tezos currency
* `bytes` : bytes sequence
* `key` : 
* `signature` : 
* `key_hash` :

## Composite types

* `*` declares a tuple made of other types.

```c
constant pair : int * string = (1, "astr")
```

* `option` declares an option of type.

```c
constant value : int option = none
```

* `list` declares a list of any type \(builtin or composed\)

```javascript
variable vals : string list = []
```

* `asset`  declares a collection of asset and the data an asset is composed of. For example the following declares a collection of real estates described by an address, a location and an owner:

```yaml
asset real_estate identified by addr {
  addr : string;
  xlocation : string;
  ylocation : string;
  owner : address;
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

```ocaml
variable c : color = Green
```

* `contract` declares the signature of another existing contract to call in entries.

```c
contract called_contract_sig {
   entry set_value (n : int)
   entry add_value (a : int, b : int)
}
```

It is then possible to declare a contract value of type _called\_contract\_sig_ and set its address:

```c
constant c : called_contract_sig = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm
```

* `aggregate` declares an asset field as a aggregate of another asset.

```yaml
asset an_asset identified by id {
  id : string;
  a_val : int;
  asset_col : another_asset aggregate;
}
```

* `partition` declares an asset field as a collection of another asset. The difference with `aggregate` is that a partition ensures at compilation that every _partitioned_ asset \(i.e. element of the partition\) belongs to one and only _partitioning_ asset.

```yaml
asset partitioning_asset identified by id {
  id : string;
  asset_part : partitioned_asset partition;
}
```

As a consequence of the partition, a _partitioned_ asset cannot be straightforwardly added or removed to its global collection with `add` and `remove` \(see operation below\). This has to be done via a partition field:

```yaml
my_partitioning_asset.asset_part.add(a_new_partitioned_asset)
```

## Effect

An entry's effect is composed of instructions separated by a semi-colon.

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

```c
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

* `isnone`

```javascript
var b_issome : bool = isnone(none);
```

* `issome`

```javascript
var b_issome : bool = issome(some(1));
```

* `getopt`

```javascript
var res : int = getopt(some(1));
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

### List

* `contains`

```javascript
archetype expr_list_contains

variable res : bool = false

entry exec () {
  specification {
    s0: res = true;
  }
  effect {
    var l : string list = ["1"; "2"; "3"];
    res := contains(l, "2")
  }
}
```

* `count` 

```javascript
archetype expr_list_count

variable res : int = 0

entry exec () {
  specification {
    s0: res = 3;
  }
  effect {
    var l : string list = ["1"; "2"; "3"];
    res := count(l)
  }
}
```

* `nth` 

```javascript
archetype expr_list_nth

variable res : string = ""

entry exec () {
  specification {
    s0: res = "2";
  }
  effect {
    var l : string list = ["1"; "2"; "3"];
    res := nth(l, 1)
  }
}
```

* `prepend` adds an element to a list at the first position. 

```javascript
archetype expr_list_prepend

variable res : string list = []

entry exec () {
  specification {
    s0: res = ["0"; "1"; "2"; "3"];
  }
  effect {
    var l : string list = ["1"; "2"; "3"];
    res := prepend(l, "0");
  }
}
```

### Asset Collection

* `contains` 

```javascript
archetype expr_method_asset_contains

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

variable res : bool = false

entry exec () {
  specification {
    s0: res = true;
  }
  effect {
    res := my_asset.contains("id0")
  }
}
```

* `count` 

```javascript
archetype expr_method_asset_count

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

variable res : int = 0

entry exec () {
  specification {
    s0: res = 3;
  }
  effect {
    res := my_asset.count()
  }
}
```

* `nth` 

```javascript
archetype expr_method_asset_nth

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

variable res : int = 0

entry exec () {
  specification {
    s0: res = 1;
  }
  effect {
    var a = my_asset.nth(1);
    res := a.value
  }
}
```

* `head` 

```javascript
archetype expr_method_asset_head

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 4};
  {"id1"; 3};
  {"id2"; 2}
}

variable res : int = 0

entry exec () {
  specification {
    s0: res = 3;
  }
  effect {
    var l = my_asset.head(2);
    var a = l.nth(1);
    res := a.value
  }
}
```

* `tail` 

```javascript
archetype expr_method_asset_tail

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

variable res : int = 0

entry exec () {
  specification {
    s0: res = 1;
  }
  effect {
    var l = my_asset.tail(2);
    var a = l.nth(0);
    res := a.value
  }
}
```

* `select` 

```javascript
archetype expr_method_asset_select

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

variable res : int = 0

entry exec () {
  specification {
    s0: res = 2;
  }
  effect {
    var l = my_asset.select(the.id = "id2");
    var a = l.nth(0);
    res := a.value
  }
}
```

* `sort` 

```javascript
archetype expr_method_asset_sort

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 3};
  {"id1"; 2};
  {"id2"; 1}
}

variable res : int = 0

entry exec () {
  specification {
    s0: res = 1;
  }
  effect {
    var l = my_asset.sort(value);
    var a = l.nth(0);
    res := a.value
  }
}
```

you can also sort with several criteria

```javascript
archetype multi_sort

asset my_asset identified by id {
  id : string;
  v1 : int;
  v2 : int;
  v3 : int;
} initialized by {
  {"id0"; 1; 2; 7};
  {"id1"; 1; 3; 9};
  {"id2"; 1; 3; 8};
  {"id3"; 1; 2; 6}
}

entry exec () {

  effect {
    var res = my_asset.sort(v1, asc(v2), desc (v3))
    (* res = ["id0"; "id3", "id1", "id2"] *)
  }
}
```

* `sum` 

```javascript
archetype expr_method_asset_sum

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

variable res : int = 0

entry exec () {
  specification {
    s0: res = 3;
  }
  effect {
    res := my_asset.sum(value)
  }
}
```

## Verification

### Specification

Here is a full example with all sections of an entry specification

```javascript
archetype contract_with_full_specification

asset myasset {
  id: string;
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

```javascript
definition mydef {
  x : myasset | forall y in col1, x.id = y.id1
}
```

* `predicate` 

```javascript
predicate mypredicate (a : int) {
  forall x in col1, forall y in col2, x.id1 = y.id2
}
```

* `variable` 

```javascript
variable myvar : int = 0
```

* `shadow effect` 

```javascript
shadow effect {
  myvar := 3
}
```

* `assert` 

```javascript
assert a1 {
  x = y
  invariant for myloop {
    x = 0;
    y = 0
  }
}
```

* `invariant` 

```javascript
invariant for myloop {
  x = 0;
  y = 0
}
```

* `postcondition` 

```javascript
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

```javascript
contract invariant c1 {
  true <> false
}
```

### Expression

All expressions in effect are available in formula part.

* `forall` universal quantifier

```javascript
q1: forall x : int, x = x;
q2: forall x in my_asset, x.value = x.value;
```

* `exists` existential quantifier

```javascript
q3: exists x : int, x = x;
q4: exists x in my_asset, x.value = x.value;
```

* `->` imply

```javascript
o1: true -> true;
```

* `<->` equivalence

```javascript
o2: true <-> true;
```

### Asset expression

* `subsetof` 

```javascript
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

```javascript
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

```javascript
s1: before.my_asset.isempty();
```

* `at` 

```javascript
s2: at(lbl).my_asset.isempty();
```

* `unmoved` 

```javascript
s3: unmoved.my_asset.isempty();
```

* `added` 

```javascript
s4: added.my_asset.isempty();
```

* `removed` 

```javascript
s5: removed.my_asset.isempty();
```

* `iterated` 

```javascript
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

```javascript
entry exec () {
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

```javascript
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

```javascript
s00 : only_by_role (anyentry, admin);
```

* `only_in_entry` 

```javascript
s01 : only_in_entry (anyentry, exec);
```

* `only_by_role_in_entry` 

```javascript
s02 : only_by_role_in_entry (anyentry, admin, exec);
```

* `not_by_role` 

```javascript
s03 : not_by_role (anyentry, admin);
```

* `not_in_entry` 

```javascript
s04 : not_in_entry (anyentry, exec);
```

* `not_by_role_in_entry` 

```javascript
s05 : not_by_role_in_entry (anyentry, admin, exec);
```

* `transferred_by` 

```javascript
s06 : transferred_by (anyentry);
```

* `transferred_to` 

```javascript
s07 : transferred_to (anyentry);
```

* `no_storage_fail` 

```javascript
s08 : no_storage_fail (anyentry);
```

