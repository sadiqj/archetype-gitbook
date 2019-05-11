---
description: When multi signature is necessary
---

# Signed by all

This extension provides an approval condition to trigger a transaction: each role in the roles argument must call the transaction signed in order to be able to call the target transaction.

{% code-tabs %}
{% code-tabs-item title="signed\_by\_all.arlx" %}
```ocaml
archetype extension signedbyall (

  action[%signedbyall roles] aTransaction

) = {

  asset signer_<%aTransaction> = {
    signer    : address;
    hasSigned : bool = false;
  } initialized by roles

  action sign_<%aTransaction> = {
    effect {
      signer.update caller {hasSigned = true}
    }
  }

  action aTransaction = {
    require {
      requires;
      <%aTransaction>_cond :
       (signer_<%aTransaction>.select(hasSigned = true)).count()
          = signer_<%aTransaction>.count()
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

