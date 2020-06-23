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

## Collection

Declaring an asset automatically creates a collection of these assets. This collection is named after the asset. In the example above,  `car` refers to the collections of car assets. In a collection, each asset has a unique identifier.

The following presents the services offered by an asset collection:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Operator</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">May fail ?</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">[ ]</td>
      <td style="text-align:left">
        <p>access asset by its key:</p>
        <p><code>car[&quot;YS3ED48E5Y3070016&quot;].model</code> is the model field
          of the car asset with id &quot;YS3ED48E5Y3070016&quot;.</p>
      </td>
      <td style="text-align:left">fails when key not in collection</td>
    </tr>
    <tr>
      <td style="text-align:left">count</td>
      <td style="text-align:left">returns the number of assets: <code>car.count()</code>
      </td>
      <td style="text-align:left">no</td>
    </tr>
    <tr>
      <td style="text-align:left">nth</td>
      <td style="text-align:left">returns the nth asset&apos;s key. <code>car.nth(0)</code> is the car asset
        with the first identification in alphabetical order.</td>
      <td style="text-align:left">fails when index out of bound.</td>
    </tr>
  </tbody>
</table>

By default, assets are sorted on the identifier value.

The type of a field may be a basic type, or one of `subset` or `partition`. 

A collection is the generic name for an asset container. For example, if `part` is the asset for vehicle part, it is possible to describe which parts a car is made of:

```ocaml
asset part identified by pid {
   pid : string;
}

asset car identified by vin {
  vin : string;
  model : string;
  year : int;
  parts : part subset;
}
```

Say that a contract deals with animals and country fields. An animal is in one, and only one field \(obviously\). A fields has a collection of animals. But rather than `collection`, the `partition` keyword specifies that _every_ animal is in _one_ field:

```ocaml
asset animal identified by aid {
  aid : string
}

asset field identified by fid {
  fid : string;
  animals : animal partition
}
```

Declaring an asset implicitly creates the collection which contains asset instances. It means that data storage in Archetype is abstract, like SQL, compared to programming languages which provides set, map, list, ... containers. 

In order to get a technical vision of asset collections, you may consider that the asset collection is equivalent to a map between the identifier field and a record made of the other fields.

For example, the Ocaml equivalent of the `car` asset declaration is a record declaration plus a map declaration.

```ocaml
type car = {
  model : string;
  year : int;
}

module CarCollection = Map.make(String)
```

Storage of asset is abstract in Archetype in order for the transcoder to apply optimization strategy when generating execution code. For example, if the car collection is only accessed via get and set methods, then it will be generated as a _set_ container.



