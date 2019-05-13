---
description: Formalise contract properties
---

# Contract specification

## Formal property

A formal property is a _logical formula_ about the relations between data values and/or asset collections properties. It is made of:

* logical connectors: and, or, implies, not, if then else, ...
* quantifiers: the universal quantifier _forall_ and the existential quantifier _exists_
* atomic predicates, which are the relations between values and/or asset collection:
  *  between values \(integers, tez, ...\): =, &lt;=, &gt;=, &lt;, &gt;, ...
  * between asset collections: subset of, ...
  * between an asset and a collection: member of, ... 

For example, say you are dealing with an auction contract which has to decide the maximum bid over all bids stored in the `bid` asset collection, and say this value is stored in the `max_bid` variable over.  The following is the way to express that `max_bid` is the highest bid:

```ocaml
forall b : bid, b.value <= max_bid
```

It reads that any stored bid value is less or equal than max\_bid.

This property would typically serve as formal specification of the function `declare_winner` which computes `max_bid` as follows:

```ocaml
variable max_bid tez = 0tz

action declare_winner = {
  specification {
     p : forall b : bid, b.value <= max_bid
  }
  effect {
     max_bid := ... (* code to compute max_bid *)
  }
}
```

This generates a verification task which consists in proving that the effect of the `declare_winner` action on the `max_bid` is actually to set it to the maximum bid value. 

{% hint style="info" %}
To practice formalisation of logical properties, you can solve online [edukera](https://app.edukera.com) formalisation exercises.
{% endhint %}

## Data invariant

It is possible to specify the property a data is supposed to have throughout the life of the contract, regardless of the calls made to the contract and the changes of values of other data.

For example,  say a `quantity` field of an asset `mile` should remain strictly positive. Use the with keyword to introduce the property after the asset declaration:

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

Say for example the data `amount` must increase due to the effect of action `add_amount`. The keyword _`before`_ is used to refer to the data value before action effect on the data. Hence this property is formalised:

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





### Assert

## Data access predicates

proof of trace



