---
description: Formalise contract properties
---

# Contract specification

## Properties

A property is a _logical formula_ about the state of the contract's storage data \(variable, asset collections, ...\). It is made of:

* logical connectors: `and`, `or`, `->` \(logical implication\), `not`, ...
* quantifiers: the universal quantifier `forall` and the existential quantifier `exists`
* atomic predicates, which are the relations between values and/or asset collection:
  * between values \(integers, tez, ...\): `=`, `<=`, `>=`, `<`, `>`, ...
  * between asset collections: `subset`, `diff`, ...
  * between an asset and a collection: `mem` \(member of\), ... 

For example, say you are dealing with an auction contract which has to decide the maximum bid over all bids stored in the `bid` asset collection, and say this value is stored in the `max_bid` variable over. The following is the way to express that `max_bid` is the highest bid:

```ocaml
forall b in bid, b.value <= max_bid
```

It reads that any stored bid value is less or equal than max\_bid.

{% hint style="info" %}
To practice the formalisation of logical properties, you can solve online [edukera](https://app.edukera.com/?qt=4) formalisation exercises.
{% endhint %}

Five kinds of properties may be distinguished:

* _postcondition_: a postcondition says something about the evolution of the storage data after the execution of an entry point
* _contract invariant_: an invariant is a property about the contract storage that is true before and after any call to entry points \(asset invariant are a special case of contract invariant\)
* _loop invariant_: a loop invariant is an invariant property applied to the body of a loop instruction: it says what stays true throughout the iteration process
* _exceptional property_: an exceptional property says something about the storage data at failing point
* _assert_: an assert property is a property local to an execution point; as such it may relate local execution variables as well as storage data

## Postconditions

A contract entry may specify postconditions. A postcondition is typically a property about the relation between the contract's storage _before_ and _after_ the execution of the entry point.

### Before and after

Say for example the variable `amount` must increase due to the effect of the entry `add_amount`. The keyword _`before`_ is used to refer to the variable value before the effect of the action. Hence this property is formalized:

```ocaml
before.amount < amount
```

This property is placed in the `specification` section of the entry:

```javascript
variable amount : int = 0

action add_amount () {
  specification {
    p : before.amount < amount
  }
  effect {
    /* do something to increase amount */
  }
}
```

In why3, this will generate a verification task to verify that the value of `amount` value after executing `add_amount` is greater than the value of `amount` before.

For asset collection, archetype provides dedicated keywords to refer to _`added`_ and _`removed`_ assets by entry effect.

Say for example you want to express that _only_ obsolete `goods` assets may have been removed by the action effect, where obsolete means with an `expiration` date is before now. This property is formalized:

```ocaml
forall x in removed.goods, x.expiration < now
```

The schema below illustrates the three sets of assets resulting from the action effect:

![action effect on asset collection](../.gitbook/assets/effect-asset.png)

This property could typically serve as a postcondition of the function `remove_obsolete`:

```javascript

entry remove_obsolete () {
  specification {
     p : forall x in removed.goods, x.expiration < now
  }
  effect {
    /* effect to remove obsolete ... */
  }
}
```

### Loop invariant

With imperative languages like archetype, iterations are done with loops. For formal verification purpose it is necessary to provide _loop invariant_ properties.

A loop invariant is a property that is true at any step of the iteration. More precisely, the invariant holds:

* at the beginning of the loop, when no object has been iterated yet \(_initialization_\)
* at each step of the iteration \(_conservation_\)
* at the end of the iteration \(_terminaison_\)

A loop invariant usually depends on the iterated assets, or on the assets still to iterate. Specific keywords are dedicated to these views of asset:

* `toiterate` refers to the assets still to iterate on
* `iterated` refers to the assets already iterated 

For example say you want to prove that the `stock` value is equal to 0 after iterating over the collection of `goods`. The loop invariant states that `stock` is upper-bounded by the sum of the `quantity` value over the `goods` assets _stil to iterate_. The following snippet illustrates how to declare this property as the loop invariant \(line 15\):

```javascript
variable stock : int = 0

asset goods {
   id       : string;
   quantity : int;
} with {
  inv : stock = goods.sum(quantity)
}

entry empty_stock () {
  specification {
    postcondition p {
       /* the effect of empty_stock is to set 'stock' to zero */
       stock = 0   
       invariant for goods_loop {
          0 <= stock <= toiterate.goods.sum(quantity)
       }
    }
  }
  effect {
    /* ... */ 
    for : goods_loop g in goods do
       /* ... decrease stock somehow */
    done;
    /* ... */
  }
}
```

Note the `invariant` section for the loop labeled `goods_loop` at line 22.

At the end of the iteration, the `toiterate` view is empty, and the loop invariant reduces to:

```ocaml
0 <= stock <= 0
```

which ensures the postcondition `p` \(line 12\). The loop invariant generates verification tasks on its own.

{% hint style="info" %}
Loop invariants may refer to local variables declared in the effect section.
{% endhint %}

{% hint style="info" %}
Archetype ensures iteration terminaison by design.
{% endhint %}

## Invariants

It is possible to specify the property a data is supposed to have throughout the life of the contract, regardless of the calls made to the contract and the changes of values of other data. Such a property is called an _invariant_.

#### Asset invariant

For example, say a `quantity` field of an asset `mile` should remain strictly positive. Use the `with` keyword to introduce the property, as illustrated below:

```ocaml
asset mile identified by id {
   id : string;
   quantity : int;
} with { 
  i : quantity > 0;
}
```

This generates as many verification tasks as the number of actions/transitions in the contract. Each verification task consists in proving that the property below holds after it is executed \(it's called the _post-condition\),_ under the assumption it holds before \(called the _pre-condition_\):

```text
forall x in mile, x.quantity > 0
```

#### State invariant

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

#### Variable invariant

Variables may also be equipped with invariants.

```javascript
variable total : int = 0 with {
   i1 : 0 <= variable < 10
}
```

The variable invariant may only refer to the variable itself. When dealing with multi-variables/assets properties, a contract invariant may be used.

#### Contract invariant

More complex contract invariants may be placed in the specification section at root level.

```javascript
archetype acontract

entry main () {
  /* ... */
}

specification {
  p : /* forall ... */
}
```

## Assert

It is possible to specify a property in the effect section of an entry \(or transition\) with the `assert` instruction. An assert property is transcoded to a verification task in why3. As such, it does not generate any execution code.

Contrary to postconditions, asserts _do_ have access to local variables in the same way as a standard instructions.

```javascript
entry anentry(i : int) {
  specification {
     assert a {
        x > i
     }
  }
  effect {
    var x = i + 1;
    assert a;
  }
}
```

It is possible to refer to a variable at a specific step of execution by declaring a code label in the code.

```javascript
entry anentry(i : int) {
  specification {
     assert a {
        at(s).x > x /* at(s).x is the value of x at label declaration (here i) */
     }
  }
  effect {
     var x = i;
     label s;
     x += 1;
     assert a; 
  }
}
```

## Security predicates

In order to provide security guarantee to contract readers, it is usually useful to state which action may change an asset collection or a variable.

For example, the following specifies that only the action `add_goods` may add a `goods` asset:

```text
only_in_action (add goods, add_goods)
```

The following specifies that only the `admin` role can do any kind of storage data:

```text
only_by_role (anychange, admin)
```

The following specifies that the role `owner` does not change any data:

```text
not_by_role (anychange, owner)
```

Archetype provides the following predicates:

| predicate | description |
| :--- | :--- |


<table>
  <thead>
    <tr>
      <th style="text-align:left"><code>no_storage_fail (ACTION)</code>
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
</table>

| `only_by_role (CHANGE, ROLE)` | specifies that **only** `ROLE` can perform `CHANGE` |
| :--- | :--- |


| `only_in_action (CHANGE, ACTION)` | specifies that **only** `ACTION` can perform `CHANGE` |
| :--- | :--- |


| `only_by_role_in_action (CHANGE, ROLE, ACTION)` | specifies that only `ROLE` can perform `CHANGE` in `ACTION` |
| :--- | :--- |


| `not_by_role (CHANGE, ROLE)` | specifies that ROLE can **not** perform`CHANGE` |
| :--- | :--- |


| `not_in_action (CHANGE, ACTION)` | specifies that ACTION can **not** perform `CHANGE` |
| :--- | :--- |


| `not_by_role_in_action (CHANGE, ROLE, ACTION)` | specifies that `ROLE` can not perform `CHANGE` in `ACTION` |
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

