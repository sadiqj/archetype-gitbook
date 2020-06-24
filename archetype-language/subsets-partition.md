---
description: When asset fields are asset containers.
---

# Partitions, Aggregates

An asset field may be a container of another asset. There are two kinds of such container:

* partition
* aggregate

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

With partitions, it is not possible to modify a partitioned collection straightforwardly with the standard instructions `add` `remove` `addupdate`. Thus the following is not authorized:

```coffeescript
effect {
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
}
```

Indeed, if the above was possible, the car asset `YS3ED48E5Y3070016` would not belong to any fleet, which is contrary to the idea of partition. Thus in this situation, archetype generates the following error:

```bash
Cannot access asset collection: asset car is partitionned by field(s) (cars).
```

### Instructions

The proper way to modify the car collection is through the `cars` partition of a fleet asset which provides the following instructions:

* `add`
* `addupdate`
* `remove`
* `clear`

```bash
effect {
   fleet["f01"].cars.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
}
```

The above instruction fails if the `fleet` collection does not contain `f01` or if the `car` collection already contains `YS3ED48E5Y3070016`.

### Update operators

When updating an asset with a partition field it is possible to add or remove assets with the `+=` and `-=` operators: 

```javascript
effect {
   fleet.update("f01", { cars += [{ "2HGFG11879H508413", "explorer", 2000, 4 }] });
   fleet.update("f02", { cars -= [ "YS3ED48E5Y3070016" ] });
}
```

## Aggregate

`aggregate` is used to _reference_ some assets. For example, say that a car may be driven by several drivers; the driver asset may refer to several cars through an `aggregate`:

```coffeescript
asset car {
   vin     : string;
   model   : string;
   year    : int;
   nbdoors : int;
}

asset driver {
   id     : string;
   drives : car aggregate;  
}
```

The literal for `aggregate` is `[]`. Aggregates are built with the _keys_ of assets to reference. 

```javascript
effect {
   driver.add({ "d01"; [] }); // empty aggregate
   driver.add({ "d02"; [ "YS3ED48E5Y3070016"; "3VWCK21Y33M306146" ] });
}
```

The above fails if a key is not present in the car collection. It means that you can only add an existing asset reference in an aggregate.

### Instructions

#### add

The `add` instruction adds a key to an aggregate. It _fails_ if the base collection does not contain that key:

```javascript
effect {
   driver["f01"].drives.add("YS3ED48E5Y3070016"); // fails if car collection 
                                                  // does not contain YS3ED48E5Y3070016
}
```

#### remove

The `remove` instruction removes a reference from an aggregate. It _fails_ if the key is not in the aggregate.

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

The `removeall` instruction removes all references in the aggregate, which is empty as a result.

```javascript
effect {
   driver["f01"].drivers.removeall();
}
```

### Update operators

When updating an asset with an aggregate field, it is possible to specify whether to add or remove asset references with the `+=` and `-=` operators: 

```javascript
effect {
   driver.update("d01", { drives += [ "2HGFG11879H508413" ]);
   driver.update("d02", { drives -= [ "YS3ED48E5Y3070016" ]);
}
```

### Synchronization

Note that it is still possible to write the car collection straightforwardly with collection instructions `add` `remove` `update` `addupdate` `clear`. 

Aggregates are _not synchronized_ though, which means that it is possible to refer to an asset that does not exist anymore, as illustrated below:

```javascript
effect {
   car.add({ "YS3ED48E5Y3070016"; "mustang"; 1968; 2});
   driver["f01"].drives.add("YS3ED48E5Y3070016");
   car.remove("YS3ED48E5Y3070016"); // at this point, driver f01's drives aggregate
                                    // refers to a non existant asset 
}
```

{% hint style="info" %}
There is no guarantee that a referenced asset in the aggregate exists in the base collection.
{% endhint %}

## Synthesis

The tables below present a synthetic view of instructions and operators availability for collection, partition and aggregates.

| Instructions | Collection | Partition | Aggregate |
| :--- | :--- | :--- | :--- |
| `add` | ok | ok | ok\* |
| `update` | ok | **na** | **na** |
| `addupdate` | ok | ok | **na** |
| `remove` | ok | ok | ok |
| `removeall` | **na** | ok | ok |
| `removeif` | ok | ok | ok |

 \*  `add` for aggregate does not have the same signature though.

| View operations | Collection | Partition | Aggregate |
| :--- | :--- | :--- | :--- |
| `count` | ok | ok | ok |
| `sum` | ok | ok | ok |
| `contains` | ok | ok | ok |
| `nth` | ok | ok | ok |
| `select` | ok | ok | ok |
| `head` | ok | ok | ok |
| `tail` | ok | ok | ok |
| `sort` | ok | ok | ok |
| `for` | ok | ok | ok |
| `clear` | ok | ok | ok |

The above table illustrates that all view operations are available for collections, partitions and aggregates.

