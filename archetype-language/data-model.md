---
description: Variable and asset collection
---

# Assets

## Declaration

An asset is defined by a set of fields, one of which is the identification field. Each asset has a unique identification value over the collection of assets.

For example, the following defines a car asset:

```javascript
asset car identified by vin {
  vin     : string;
  model   : string;
  year    : int;
  nbdoors : int;
}
```

When `identified by` is omitted, the first field is the identifier. 

## Collection

Declaring an asset automatically creates a collection of these assets. This collection is named after the asset. In the example above,  `car` refers to the collections of car assets. In a collection, each asset has a unique identifier.

### Write

Use the `add` instruction to add an asset to the collection. It _fails_ if an asset with the same id is already in the collection:

```ocaml
effect {
   car.add({ "YS3ED48E5Y3070016", "mustang", 1968, 2});
}
```

Use the `update` instruction to update a field of an asset. It _fails_ if the collection does not have an asset with the id:

```javascript
effect {
   /* update one field */
   car.update("YS3ED48E5Y3070016", { year : 1967 });
   /* update severa fields */ 
   car.update("YS3ED48E5Y3070016", { year : 1967, nbdoors : 3 });
   /* update all fields, no need for field name */
   car.update("YS3ED48E5Y3070016", { "fiesta", 2020, 4 });
}
```

Use the `addupdate` instruction to add or update an update. The argument asset is either added if not already present or updated if present. It does _not_ fail.

```javascript
effect {
   // all fiels are necessary, no field name required
   car.addupdate("YS3ED48E5Y3070016", {  "mustang", 1968, 2 });
}
```

Use the `remove` instruction to remove an asset. It does _not_ fail if the collection does not have an asset with the id.

```ocaml
effect {
   car.remove("YS3ED48E5Y3070016");
}
```

Use the `clear`  instruction to remove all asset in a collection:

```javascript
effect {
   car.clear();
}
```

Use the `removeif` instruction to remove assets under given criteria. For example, in order to remove all cars with less than 4 doors:

```ocaml
effect {
  car.removeif(the.nbdoors < 4);
}
```

The `the` keyword refers to the asset being evaluated. 

### Read assets

#### by key

The `[ ]` and `.` syntax is used to retrieve an asset data and access to a field:

```ocaml
effect {
  var amodel = car["YS3ED48E5Y3070016"].model;
  ...
}
```

#### by position

`nth` returns the key at position n in the collection. Indeed assets are ordered according to the identification field. It fails if n is out of the bounds of the collection.

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016", "mustang", 1968, 2});
   car.add({ "3VWCK21Y33M306146", "fiesta", 2010, 4});
   if car.nth(0) = "3VWCK21Y33M306146" then transfer 1tz to coder; 
}
```

### Aggregate

`count` returns the number of elements in a collection:

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016", "mustang", 1968, 2});
   if car.count() = 1 then transfer 1tz to coder;
}
```

`sum` returns the sum of a field:

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016", "mustang", 1968, 2});
   car.add({ "3VWCK21Y33M306146", "fiesta", 2010, 4});
   car.add({ "1FTDF15Y2SNB02216", "focus", 2008, 4});
   if car.sum(nbdoors) = 10 then transfer 1tz to coder;
}
```

Views a collection

head

tail

select

sort





