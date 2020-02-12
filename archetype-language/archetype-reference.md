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

* `action` declares an entry point of the contract. An action has the following sections:
  * `specification` \(optional\) to provide the _post conditions_ the action is supposed to have
  * `accept transfer` \(optional\) to specify that transfer of tez is accepted 
  * `called by` \(optional\) to declare which role may call this action
  * `require` \(optional\) to list the necessary conditions for the action to be executed
  * `failif` \(optional\)to list the conditions which prevent from execution 
  * `effect` is the code to execute by the action

```css
action an_action_1 (arg1 : string, arg2 : int) {
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

* `transition` declares an entry point of the contract that changes the state of the contract. A transition has the same sections as an action \(except `effect`, see above\) plus:
  * `from` to specify the states the transition starts from
  * `to` to specify the states after the transition 
  * `when` \(optional\) to specify the transition condition
  * `with effect` to specify the effect of the transition

```c
states 
| Created initial
| Confirmed
| Canceled
| Success
| Fail

transition confirm {
  from Created
  to Confirmed
  when { transferred > 10 tz }
  with effect {
    transfer 1tz to @tz1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm
  }
}
```

It is possible to specify several `to ... when .... with effect ...` sections in a transition. It is possible to specify a list of original states for the transition to start from:

```c
transition fail {
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
* `role` : an address that can be used in action's called by section
* `date` : date values \(ISO format\)
* `duration` : duration values \(in day, week, month or year\)
* `string` : string of characters
* `bytes` : bytes sequence 
* `tez` : Tezos currency

## Composite types

* `*` declares a tuple made of other types.

```c
constant pair : int * string = 1, "astr"
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

* `contract` declares the signature of another existing contract to call in actions.

```c
contract called_contract_sig {
   action set_value (n : int)
   action add_value (a : int, b : int)
}
```

It is then possible to declare a contract value of type _called\_contract\_sig_ and set its address:

```c
constant c : called_contract_sig = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm
```

* `collection` declares an asset field as a collection of another asset.

```yaml
asset an_asset identified by id {
  id : string;
  a_val : int;
  asset_col : another_asset collection
}
```

* `partition` declares an asset field as a collection of another asset. The difference with `collection` is that a partition ensures at compilation that every _partitioned_ asset \(i.e. element of the partition\) belongs to one and only _partitioning_ asset.

```yaml
asset partitioning_asset identified by id {
  id : string;
  asset_part : partitioned_asset partition
}
```

As a consequence of the partition, a _partitioned_ asset cannot be straightforwardly added or removed to its global collection with `add` and `remove` \(see operation below\). This has to be done via a partition field:

```yaml
my_partitioning_asset.asset_part.add(a_new_partitioned_asset)
```

## Effect

An action's effect is composed of instructions separated by a semi-colon. 

* `var` declares a local variable with its value. While possible, it is not necessary to declare the type a local variable. Local variables' identifier must be unique \(no name capture in Archetype\).

```javascript
var i = 0;
```

* `:=` enables to assign a new value to a global or local variable. 

```javascript
i := 1;
```

* `+=` , `-=` , `*=` and `/=` enable to respectively increment and decrement an integer variable \(local or global\).

```javascript
i *= 2;  // i := i * 2
d += 3w; // d := i + 3w, date d is now previous d plus 3 weeks
```

* `add` enables to add an asset to an asset collection. It fails if the asset key is already present in the collection.

```javascript
an_asset.add({ id = "RJRUEQDECSTG", asset_col = [] }); // see 'collection' example
```

* `update` enables to update an existing asset; it takes as arguments the id of the asset to update and the list of effects on the asset. It fails if the id is not present in the asset collection.

```javascript
an_asset.update("RJRUEQDECSTG", { a_value += 3 });
```

* `addupdate` is similar to update except that it adds the asset if its identifier is not present in the collection.
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

* `prepend` adds an element to a list at the first position. 

```javascript
var l : string list = ["1"; "2"; "3"];
l.prepend("0"); // l is ["0"; "1"; "2"; "3"]
```

* `if then` and `if then else` are the conditional branchings instructions. 

```ocaml
if i > 0 then (
  // (* effect when i > 0 *) 
) else (
  // (* effect when i <= 0 *)
);
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
      res += i
done;               // res is 6
```

* `transfer` transfers an amount of tez to an address or a contract.

```c
transfer 2tz to @tz1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm;
```

With the `contract` keyword presented above, it is possible to transfer to a contract and call an entry point. In the example below, the entry point `set_value` of contract `c` is called and `2tz` is transferred.

```c
contract contract_called_sig {
   action set_value (n : int)
   action add_value (a : int, b : int)
}

constant c : called_contract_sig = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm

action update_value(n : int) {
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

* `and`
* `or`
* `not`

### Operator

* `=`
* `<>`
* `<`
* `<=`
* `>`
* `>=`

### Arithmetic

* `+`
* `-`
* `*`
* `/`
* `%`

### Constant

* `sender`
* `balance`
* `transferred`
* `now`
* `source`
* `state` 

### List

* `contains`
* `count`
* `nth`

### Asset Collection

* `contains`
* `count`
* `get`
* `head`
* `max`
* `min`
* `nth`
* `select`
* `sort`
* `sum`
* `tail`
* `update` 

## Verification



  
  






