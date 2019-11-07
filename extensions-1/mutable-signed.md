---
description: Adds a signed setter
---

# Mutable signed

This extension defines a mutable value. The action to change its value must be signed by roles passed as argument.

{% tabs %}
{% tab title="mutable\_signed.arlx" %}
```ocaml
archetype extension mutable_signed (

  variable[%mutable_signed roles cond%] avariable type

) = {

  action[%signedbyall roles%] set_<%avariable> (newvariable : type) = {
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

