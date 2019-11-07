---
description: Adds a setter action for the extended variable.
---

# Mutable

This extension takes two arguments:

1. the role expression to specify who can change the value
2. the condition to change the value

{% tabs %}
{% tab title="mutable.arlx" %}
```ocaml
archetype extension mutable (

  variable[%mutable arole cond%] avariable type

) = {

  action set_<%avariable> (new<%avariable> : type) = {
    called by arole
    require {
      set_<%avariable>_c1 : cond
    }
    effect {
      avariable := newvariable
    }
  }
}
```
{% endtab %}
{% endtabs %}

