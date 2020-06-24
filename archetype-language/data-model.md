---
description: Variable and asset collection
---

# Assets

## Declaration

An asset is defined by a set of fields, one of which is the identification field. Each asset has a unique identification value over the collection of assets.

For example, the following defines a car asset:

```coffeescript
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

#### add

Use the `add` instruction to add an asset to the collection. It _fails_ if an asset with the same id is already in the collection:

```ocaml
effect {
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
}
```

#### update

Use the `update` instruction to update a field of an asset. It _fails_ if the collection does not have an asset with the id:

```javascript
effect {
   /* update one field */
   car.update("YS3ED48E5Y3070016", { year = 1967 });
   /* update severa fields */ 
   car.update("YS3ED48E5Y3070016", { year = 1967; nbdoors = 3 });
   /* update all fields, no need for field name */
   car.update("YS3ED48E5Y3070016", { "fiesta"; 2020; 4 });
}
```

#### addupdate

Use the `addupdate` instruction to add or update an update. The argument asset is either added if not already present or updated if present. It does _not_ fail.

```javascript
effect {
   // all fiels are necessary, no field name required
   car.addupdate("YS3ED48E5Y3070016", {  "mustang"; 1968; 2 });
}
```

#### remove

Use the `remove` instruction to remove an asset. It does _not_ fail if the collection does not have an asset with the id.

```ocaml
effect {
   car.remove("YS3ED48E5Y3070016");
}
```

#### clear

Use the `clear`  instruction to remove all asset in a collection:

```javascript
effect {
   car.clear();
}
```

#### removeif

Use the `removeif` instruction to remove assets under given criteria. For example, in order to remove all cars with less than 4 doors:

```ocaml
effect {
  car.removeif(the.nbdoors < 4);
}
```

The `the` keyword refers to the asset being evaluated. 

### Read an asset

The `[ ]` and `.` syntax is used to retrieve an asset data and access to a field:

```ocaml
effect {
  var amodel = car["YS3ED48E5Y3070016"].model;
  ...
}
```

## Views

Views are used to access assets in a different way than the base collection seen above. They enable access with a different order, position range, or specific criteria.

You cannot build a view from scratch. It is derived from a collection with the operators presented below. Note at last that you cannot modify the collection through a view.

### Create

#### head

`head` returns the first n elements of the collection or view. By default assets are ordered by the identification field. It _fails_ if n is greater than the number of elements in the collection or view \(or negative\).

```javascript
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.head(2); // view creation :
                        // v has 1FTDF15Y2SNB02216 and 3VWCK21Y33M306146 assets
   ...
}
```

#### tail

`tail` returns the _last_ n elements of the collection or view. By default assets are ordered by the identification field. It _fails_ if n is greater than the number of elements in the collection or view \(or negative\).

```javascript
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.tail(1); // view creation :
                        // v has only the YS3ED48E5Y3070016 asset
   ...
}
```

#### select

`select` returns a view with assets that satisfy a criterion:

```javascript
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.select(the.nbdoors = 4); // view creation
                                        // v contains 3VWCK21Y33M306146 and 
                                        // 3VWCK21Y33M306146 assets
   ...
}
```

As in `removeif` above, the `the` keyword refers to the asset being evaluated. 

#### sort

`sort` provides access to assets in a different order than the default one based on the identification order. 

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   var v = car.sort(nbdoors);
   if v.nth(0) = "YS3ED48E5Y3070016" then transfer 1tz to coder;
   if v.nth(1) = "1FTDF15Y2SNB02216" then transfer 1tz to coder;
   if v.nth(2) = "3VWCK21Y33M306146" then transfer 1tz to coder;
}
```

Note in the example below that the sort operator preserves the initial order for assets with the same criterion. In this example, `1FTDF15Y2SNB02216` comes before `3VWCK21Y33M306146`.

It is possible to specify the descending order with the `desc` operator:

```ocaml
var v = car.sort(desc(nbdoors));
```

### Test membership

`contains` returns true if the view or collection contains a key. 

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   if car.contains("YS3ED48E5Y3070016") then transfer 1tz to coder;
}
```

### Access by position

`nth` returns the key at position n in the view. By default assets are ordered by the identification field. It _fails_ if n is out of the bounds of the collection.

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   if car.nth(0) = "3VWCK21Y33M306146" then transfer 1tz to coder; 
   if car[car.nth(0)].model = "mustang" then transfer 1tz to coder;
}
```

### Aggregate

#### count

`count` returns the number of elements in a collection:

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   if car.count() = 1 then transfer 1tz to coder;
}
```

#### sum

`sum` returns the sum of a field:

```ocaml
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   car.add({ "3VWCK21Y33M306146"; "fiesta"; 2010; 4});
   car.add({ "1FTDF15Y2SNB02216"; "focus"; 2008; 4});
   if car.sum(nbdoors) = 10 then transfer 1tz to coder;
}
```

