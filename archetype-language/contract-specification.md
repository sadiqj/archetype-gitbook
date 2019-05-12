---
description: Formalise contract properties
---

# Contract specification

## Formal property

A formal property is a _logical formula_ about the relations between data values and/or asset collections properties. It is made of:

* logical connectors: and, or, implies, not
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

This property would typically serve as formal specification for the function which computes `max_bid` as follows:

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

## Data invariant

## Effect specification

### Post condition

### Loop invariant

### Assert

## Contract specification

proof of trace



