---
description: Variable and asset collection
---

# Data storage

## Constant

A constant is a global value that cannot be changed. The following snippet defines a global constant named `price` typed `tez` equal to 10 tez:

```ocaml
constant price tez = 10tz
```

The following defines a `string` typed constant named `msg` :

```ocaml
constant msg string = "Hello world"
```

## Variable

A variable is a global value that can be changed. The following snippet defines 2 global variables named `seller` and `buyer` typed role equal to a \(here random\) addresses:

```ocaml
variable seller role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk
variable buyer  role = @tz1KUbontHGor25CztkksC8RvjUWAbXYJuNr
```

The following defines a variable `price` typed `tez`  received from `buyer`, to transfer to `seller`.

```ocaml
variable price tez from buyer to seller = 100tz
```

## Asset

An archetype contract can store asset data. An asset is described with a set of fields, one of which is the asset identification field. Each asset has a unique identification value.

For example, the following defines a car asset:

```ocaml
asset car identified by vin = {
  vin : string;
  model : string;
  year : int
}
```

The type of a field may be a basic type, but can also be one of `collection` or `partition`. 

A collection is the generic name for an asset container. For example, if `part` is the asset for vehicle part, it is possible to describe which parts a car is made of:

```ocaml
asset part identified by pid = {
   pid : string
}

asset car identified by vin = {
  vin : string;
  model : string;
  year : int;
  parts : part collection
}
```

Say now that the contract deals with animals and country fields. An animal is in one, and only one field \(obviously\). A fields contains a collection of animal. Rather than `collection`, the `partition` keyword specifies that _every_ animal is in _one_ field:

```ocaml
asset animal identidied by = {
  aid : string
}

asset field identified by fid = {
  fid : string;
  animals : animal partition
}
```

Declaring an asset implicitly creates the collection which contains asset instances. It means that data storage in Archetype is abstract, like SQL, compared to programming languages which provides set, map, list, ... containers. 

In order to get a technical vision of asset collections, you may consider that the asset collection is equivalent to a map between the identifier field and a record made of the other fields.

For example, the ocaml equivalent of the `car` asset above is a record declaration and a map declaration.

```ocaml
type car = {
  model : string;
  year : int;
}

module CarColl = Map.make(String)
```

Storage of asset is abstract in Archetype in order for the transcoder to apply optimisation strategy when generating execution code. For example, if the car collection is only accessed via get and set methods, then it will be generated as a set container.



