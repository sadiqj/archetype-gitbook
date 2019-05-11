---
description: Transfer a role you own to someone else
---

# Transferable

The transferable extension defines a basic two-step process to transfer a role value :

1. current role assigns a new address for the role
2. new address confirms the transfer

{% code-tabs %}
{% code-tabs-item title="transferable.arlx" %}
```ocaml
archetype extension transferable (

 variable[%transferable] aRole role

) = {
  variable <%aRole>_tmp role

  action assign_new_<%aRole> (newrole : role) = {
    called by aRole
    effect {
      <%aRole>_tmp := newrole
    }
  }

  action confirm_<%aRole> = {
    called by <%aRole>tmp
    effect {
      aRole := <%aRole>tmp
    }
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

