---
description: Requires multi-signature
---

# Signed by all

This extension provides an approval condition to trigger a transaction: each role in the _roles_ argument must call the _sign_ transaction in order to be able to call the signed transaction.

It is very likely that this extension be soon promoted to a native _action_ instruction.

{% code-tabs %}
{% code-tabs-item title="signed\_by\_all.arlx" %}
```ocaml
archetype extension signedbyall (

  action[%signedbyall roles%] aTransaction

) = {

  asset signer_<%aTransaction> identified by id_<%aTransation> = {
    id_<%aTransaction> : address;
  }

  action sign_<%aTransaction> = {
    called by roles   (* do not forget this *)
    effect {
      signer_<%aTransaction>.add { id_<%aTransaction> = caller }
    }
  }

  action aTransaction = {
    require {
      requires;
      <%aTransaction>_signedbyall : 
         signer_<%aTransaction>.count() = roles.count()
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

