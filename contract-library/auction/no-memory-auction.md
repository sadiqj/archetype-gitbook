---
description: Most basic auction process
---

# No memory

This is the most basic version of an auction process. It does not memorise who bid what.

{% code-tabs %}
{% code-tabs-item title="auction\_no\_memory.arl" %}
```ocaml
archetype auction_no_memory

variable bid tez = 0tz
variable incumbent address = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk

variable deadline date = 2019-01-01T00:00:00

transaction place_bid (id : address) (b : tez) = {
  require { 
    c1 : now < deadline;
    c2 : b > bid
  }
   
  effect {
    incumbent := caller;
    bid := transferred
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}



