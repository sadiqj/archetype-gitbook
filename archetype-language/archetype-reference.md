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
  * `called` by \(optional\) to declare which role may call this action
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

* `transition` declares an entry point of the contract that changes the state of the contract. A transition has the following sections:

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

* `asset`  declares a collection of asset and the data an asset is composed of. For example the following declares a collection of real estates described by an address, a location and an owner:

```yaml
asset real_estate identified by addr {
  addr : string;
  xlocation : string;
  ylocation : string;
  owner : address;
}
```

* `enum`  is used to declare an enumeration value. It is read with a `match ... with` command \(see below\). 

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

* `contract` is used to declare the signature of another existing contract to call in actions.

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



