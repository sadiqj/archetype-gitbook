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

The literal for `subset` is `[]`. Subsets are built with the _keys_ of assets to reference. 

```javascript
effect {
   driver.add({ "d01"; [] }); // empty subset
   driver.add({ "d02"; [ "YS3ED48E5Y3070016"; "3VWCK21Y33M306146" ] });
}
```

The above fails if a key is not present in the car collection. It means that you can only add an existing asset reference in a subset.

### Instructions

#### add

The `add` instruction adds a key to a subset. It _fails_ if the base collection does not contain that key:

```javascript
effect {
   driver["f01"].drives.add("YS3ED48E5Y3070016"); // fails if car collection 
                                                  // does not contain YS3ED48E5Y3070016
}
```

#### remove

The `remove` instruction removes a reference from a subset. It _fails_ if the key is not in the subset.

```javascript
effect {
   car.clear();
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   driver["f01"].drives.add("YS3ED48E5Y3070016");
   // now remove it
   driver["f01"].drivers.remove("YS3ED48E5Y3070016");
   // this does not remove it from the car collection
   if car.contains("YS3ED48E5Y3070016") then transfer 1tz to coder;
}
```

#### removeall

The `removeall` instruction removes all references in the subset, which is empty as a result.

```javascript
effect {
   driver["f01"].drivers.removeall();
}
```

### Synchronization

Note that it is still possible to write the car collection straightforwardly with collection instructions `add` `remove` `update` `addupdate` `clear`. 

Subsets are _not synchronized_ though, which means that it is possible to refer to an asset that does not exist anymore, as illustrated below:

```javascript
effect {
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   driver["f01"].drives.add("YS3ED48E5Y3070016");
   car.remove("YS3ED48E5Y3070016"); // at this point, driver f01's drives subset
                                    // refers to a non existant asset 
}
```

{% hint style="info" %}
In this version of archetype, there is no guarantee of the existence of a reference in a subset. 
{% endhint %}

## Synthesis

The table below presents synthetic view 





 





