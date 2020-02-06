# Reference

An archetype contract is composed of declarations, actions \(aka entry points\) and optionally specification for formal verification.

## Declarations

* `constant` and `variables` keywords are used to declare global variables. A constant value cannot be modified in the contract's actions.

```ocaml
constant rate rational = 0.2
```

```ocaml
variable price tez = 10tz
```

* `states` keyword is used to declare contract possible states. It is then possible to use _transitions_ to change the contract states \(see below\)

```ocaml
states =
| Created initial
| Confirmed
| Canceled
| Success
| Fail
```

 `initial`  is used to declare the initial state value of the contract. 

* `asset` is used to declare a collection of asset and the data an asset is composed of. For example the following declares a collection of real estates described by an address, a location and an owner:

```yaml
asset real_estate identified by addr {
  addr : string;
  xlocation : string;
  ylocation : string;
  owner : address;
}
```

* action is used to declare 

## Composite types

* `enum` keyword is used to declare an enumeration value. It is read with a `match ... with` command \(see below\). 

```ocaml
enum color =
| White 
| Red
| Green
| Blue
```

It is then possible to declare a variable of type _color_:

```ocaml
variable c color = Green
```

* `contract` is used to declare the signature of another existing contract to call in the actions.

