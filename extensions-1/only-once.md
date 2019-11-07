---
description: Specifies an action be executed only once
---

# Only once

This extension is anchored at action level; it adds the specification that the function, if called several times, behaves as if it is called once. It is typically used for withdraw transactions.

It is implemented with a simple function’s action execution counter.

{% tabs %}
{% tab title="only\_once.arlx" %}
```ocaml
archetype extension onlyonce (

  action[%onlyonce%] tr

) = {

  action tr = {
    verification {
      variable call_count int = 0
      effect {
        call_count += 1
      }
      invariant loop = {
        i1 : requires -> call_count = 0;
        i2 : call_count <= 1
      }
      specification {
        none
      }
    }
  }
}
```
{% endtab %}
{% endtabs %}

Variables and action declared in specification are mapped to why3 ghost code. The invariant section is mapped to why3 storage type \(record\) invariants.

`requires` is a keyword that stands for the conjunction of conditions declared in the transaction: if c1 and c2 are the conditions of the transaction, conditions stands for c1 and c2.

