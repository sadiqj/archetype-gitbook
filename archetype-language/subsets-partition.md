---
description: When asset fields are asset containers.
---

# Subset, Partition

An asset field may be a container of another asset. There are two kinds of such container:

* subset
* partition

## Partition

A partition is used when an asset _belongs_ to one and exactly one other asset. For example, say that the collection of cars is organized in different fleets so that one car asset belongs to only one fleet asset:

```coffeescript
asset car {
   vin     : string;
   model   : string;
   year    : int;
   nbdoors : int;
}

asset fleet {
   id   : string;
   cars : car partition;  # a car asset belongs to one fleet asset
}
```

The literal for partition is `[]`.

```javascript
effect {
   fleet.add({ "f01"; []}); // empty partition
   fleet.add({ "f02"; [{ "YS3ED48E5Y3070016"; "mustang"; 1968; 2}]);
}
```

With partitions, it is not possible to modify a partitioned collection straightforwardly with the standard instructions `add` `remove` `addupdate` `clear`. The following is not authorized:

```coffeescript
effect {
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
}
```

Indeed, if the above was possible, the car asset `YS3ED48E5Y3070016` would not belong to any fleet, which is contrary to the idea of partition. Thus in this situation, archetype generates the following error:

```bash
Cannot access asset collection: asset car is partitionned by field(s) (cars).
```

The proper way to modify the car collection is through a fleet asset and the `cars` partition which provides the following instructions:

* `add`
* `remove`
* `udpate`
* `addupdate`
* `clear`

```bash
effect {
   fleet["f01"].cars.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
}
```

The above instruction fails if the `fleet` collection does not contain `f01` or if the `car` collection already contains `YS3ED48E5Y3070016`.

## Subset

A subset is used to _reference_ some assets. For example, say that a car may be driven by several drivers; the driver asset may refer to several cars through a `subset`:

```coffeescript
asset car {
   vin     : string;
   model   : string;
   year    : int;
   nbdoors : int;
}

asset driver {
   id     : string;
   drives : car subset;  
}
```

The literal for subset is `[]`. Subset are built with the _keys_ of assets to reference. 

```javascript
effect {
   driver.add({ "d01"; [] }); // empty subset
   driver.add({ "d02"; [ "YS3ED48E5Y3070016"; ""; "" ] });
}
```





