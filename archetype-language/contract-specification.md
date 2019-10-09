---
description: Formalise contract properties
---

# Contract specification

## Formal property

A formal property is a _logical formula_ about the relations between data values and/or asset collections properties. It is made of:

* logical connectors: `and`, `or`, `->` \(logical implication\), `not`, ...
* quantifiers: the universal quantifier `forall` \_\_and the existential quantifier `exists`
* atomic predicates, which are the relations between values and/or asset collection:
  * between values \(integers, tez, ...\): `=`, `<=`, `>=`, `<`, `>`, ...
  * between asset collections: `subset`, `diff`, ...
  * between an asset and a collection: `mem` \(member of\), ... 

For example, say you are dealing with an auction contract which has to decide the maximum bid over all bids stored in the `bid` asset collection, and say this value is stored in the `max_bid` variable over. The following is the way to express that `max_bid` is the highest bid:

```ocaml
forall b : bid, b.value <= max_bid
```

It reads that any stored bid value is less or equal than max\_bid.

This property would typically serve as formal specification of the function `declare_winner` which computes `max_bid` as follows:

```ocaml
variable max_bid tez = 0tz

action declare_winner = {
  specification {
     p : forall b : bid, b.value <= max_bid;
  }
  effect {
     max_bid := ... (* code to compute max_bid *)
  }
}
```

This generates a verification task which consists in proving that the effect of the `declare_winner` action on the `max_bid` is actually to set it to the maximum bid value.

{% hint style="info" %}
To practice formalisation of logical properties, you can solve online [edukera](https://app.edukera.com/?qt=4) formalisation exercises.
{% endhint %}

## Data invariant

It is possible to specify the property a data is supposed to have throughout the life of the contract, regardless of the calls made to the contract and the changes of values of other data.

For example, say a `quantity` field of an asset `mile` should remain strictly positive. Use the `with` keyword to introduce the property, as illustrated below:

```ocaml
asset mile identified by id = {
   id : string;
   quantity : int;
} with { 
  i : quantity > 0
}
```

This generates as many verification tasks as the number of actions/transitions in the contract. Each verification task consists in proving that the property below holds after it is executed \(it's called the _post-condition\),_ under the assumption it holds before \(called the _pre-condition_\):

```text
forall x : mile, x.quantity > 0
```

It is also possible to declare a state invariant.

Say for example the contract should not hold any currency in the state `Terminated`. The following would generate the necessary verification tasks:

```ocaml
states =
| Created initial
| Confirmed
| Canceled
| Terminated with { i : balance = 0 }
```

This generates the following pre and post condition for all actions and transitions:

```ocaml
if state = Terminated then balance = 0
```

## Effect specification

### Before and after

When formalising action property, it is usually necessary to express that the value of a storage data _after_ the execution of the action has a certain relation towards the same value _before_ the execution.

Say for example the variable `amount` must increase due to the effect of action `add_amount`. The keyword _`before`_ is used to refer to the variable value before the effect of the action. Hence this property is formalised:

```ocaml
before amount < amount
```

This property is placed in the `specification` section of the action:

```ocaml
variable amount int = 0

action add_amount = {
  specification {
    p : before amount < amount
  }
  effect {
    (* do something to increase amount *)
  }
}
```

For asset collection, archetype provides dedicated keywords to refer to _`added`_, _`removed`_ and _`unmoved`_ assets by action effect.

Say for example you want to express the _only_ obsolete `goods` assets may have been removed by the action effect, that is asset with `expiration` date before now. This property is formalised:

```ocaml
forall x : removed goods, x.expiration < now
```

The schema below illustrates the three sets of assets resulting from the action effect:

![action effect on asset collection](../.gitbook/assets/effect-asset.png)

### Loop invariant

With imperative languages like archetype, iterations are done with loops. For formal verification purpose, it is necessary to provide _loop invariant_ properties.

A loop invariant is a property which is true during iteration. More precisely, the invariant holds:

* at the beginning of the loop, when no object has been iterated yet \(_initialisation_\)
* at each step of the iteration \(_conservation_\)
* at the end of the iteration \(_terminaison_\)

  A loop invariant usually depends on the already iterated assets, or on the assets stil to iterate. Specific keywords are dedicated to these asset collections:

* `toiterate` refers to the assets stil to iterate on
* `iterated` refers to the assets already iterated 

For example say you want to prove that the `stock` value is equal to 0 after iterating over the collection of `goods`. The loop invariant states that `stock` is upper-bounded by the sum of the `quantity` value over the `goods` assets _stil to iterate_. The following snippet illustrates how to declare this property as the loop invariant \(line 15\):

```ocaml
variable stock int = 0

asset goods identified by id= {
   id : string
   quantity : int
} with {
  a : stock = goods.sum(quantity)
}

action empty_stock = {
  specification {
    postcondition p = {
       stock = 0   (* this specifies that the effect of empty_stock is to
                      set 'stock' to zero *)
       invariant for goods_loop {
          i : 0 <= stock <= toiterate goods.sum(quantity)
       }
    }
  }
  effect {
    ... 
    for : goods_loop (g in goods) {
       ... (* decrease stock somehow *)
    }
    ...
  }
}
```

Note the `invariant` section for the loop labeled `goods_loop` at line 13.

At the end of the iteration, the `toiterate` collection is empty, and the loop invariant reduces to:

```ocaml
0 <= stock <= 0
```

which ensures post condition `p` \(line 12\). The loop invariant generates verification tasks on its own.

{% hint style="info" %}
Loop invariants may refer to local variables declared in the effect section.
{% endhint %}

{% hint style="info" %}
Archetype ensures iteration terminaison by design.
{% endhint %}

### Assert

As in many languages, it is possible to insert `assert` instructions in the effect of actions and transitions.

The difference in archetype is that an assert instruction generates a verification task, which means that it is verified _before_ the contract is deployed.

As such, an `assert` instruction does not generate any execution code.

## Security predicates

In order to provide security guarantee to contract readers, it is usually useful to state which action may change an asset collection or a variable.

For example, the following specifies that only the action `add_goods` may add a `goods` asset:

```text
only_in_action (add goods) add_goods
```

The following specifies that only the `admin` role can do any kind of storage data:

```text
only_by_role anychange admin
```

The following specifies that the role `owner` does not change any data:

```text
not_by_role anychange owner
```

Archetype provides the following predicates:

| predicate | description |
| :--- | :--- |


<table>
  <thead>
    <tr>
      <th style="text-align:left"><code>no_storage_fail ACTION</code>
      </th>
      <th style="text-align:left">
        <p>specifies that <code>ACTION</code>  <b>cannot</b> logically fail
          <br />on storage access (key not found when</p>
        <p>reading and setting, and key already
          <br />present when adding an asset)</p>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>| `only_by_role CHANGE ROLE` | specifies that **only** `ROLE` can perform `CHANGE` |
| :--- | :--- |


| `only_in_action CHANGE ACTION` | specifies that **only** `ACTION` can perform `CHANGE` |
| :--- | :--- |


| `only_by_role_in_action CHANGE ROLE ACTION` | specifies that only `ROLE` can perform `CHANGE` in `ACTION` |
| :--- | :--- |


| `not_by_role CHANGE ROLE` | specifies that ROLE can **not** perform`CHANGE` |
| :--- | :--- |


| `not_in_action CHANGE ACTION` | specifies that ACTION can **not** perform `CHANGE` |
| :--- | :--- |


| `not_by_role_in_action CHANGE ROLE ACTION` | specifies that `ROLE` can not perform `CHANGE` in `ACTION` |
| :--- | :--- |


{% tabs %}
{% tab title="ROLE" %}
* a variable typed `role`  
* a list of `ROLE` separated by `or`
{% endtab %}

{% tab title="CHANGE" %}
* `add ASSET` 
* `remove ASSET` 
* `update (ASSET FIELD | VARIABLE)` 
* a list of above `CHANGE` separated by `or`
* `transfers`
* `transfer to ROLE`
* _`anychange`_
* _`anychange but`_ `ACTION`
* _`nochange`_
{% endtab %}

{% tab title="ACTION" %}
* a contract action name
* a list of `ACTION` separated by `or`
* _`anyaction`_
* _`anyaction but`_ `ACTION`
* _`noaction`_
{% endtab %}
{% endtabs %}

