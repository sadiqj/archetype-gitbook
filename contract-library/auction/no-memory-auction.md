---
description: Most basic auction process
---

# No memory

This is the most basic version of an auction process. It does not memorise who bid what.

{% tabs %}
{% tab title="auction\_no\_memory.arl" %}
```ocaml
archetype auction_no_memory

variable bid mtez = 0mtz
variable incumbent address

variable deadline date = 2019-01-01T00:00:00

action place_bid (id : address) (b : mtez) = {
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
{% endtab %}
{% endtabs %}

