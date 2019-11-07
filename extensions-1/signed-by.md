---
description: Requires an action argument be signed
---

# Signed by

## Signed argument

This extension applies to an action field; it adds to the action the certificate that the argument comes from the role passed as the argument.

{% tabs %}
{% tab title="signedby.arlx" %}
```ocaml
archetype extension signedby (
  
  action aTransaction (anarg[%signedby arole%] : type)

) = {
   
  variable key_<%arole> key = oracle

  action aTransaction (signed_<%anarg> : signature) = {
    require {
      c1 : check_signature anarg signed_<%anarg> key_<%arole>
    }
  }

}
```
{% endtab %}
{% endtabs %}

## Signed Object

This extension applies to an object and adds to the action the certificate that the argument comes from the role passed as the argument.

{% tabs %}
{% tab title="object\_signed\_by.arlx" %}
```ocaml
archetype extension signedby_at_object_level (
  
  variable[%signedby_at_object_level arole%] anObject object

  action anAction (obj : anObject)

) = {
   
  variable key_<%arole> key = oracle

  action anAction (signed_<%obj> : signature) = {
    require {
      c1 : check_signature obj signed_<%obj> key_<%arole>
    }
  }

}

```
{% endtab %}
{% endtabs %}

