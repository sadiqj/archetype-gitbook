---
description: Most basic auction process
---

# No memory

This is the most basic version of an auction process. It does not memorise who bid what.

{% code title="auction\_no\_memory.arl" %}
```ocaml
archetype auction_no_memory

variable bid : tez = 0tz
variable incumbent : address = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

variable deadline : date = 2019-01-01T00:00:00

action place_bid (id : address, b : tez) {
   require {
      c1 : now < deadline;
      c2 : b > bid;
   }

   effect {
      incumbent := caller;
      bid := transferred
   }
}

```
{% endcode %}

