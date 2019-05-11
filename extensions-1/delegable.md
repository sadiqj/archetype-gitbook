---
description: >-
  This extension factorises the delegation mechanism found in the ERC721
  contract.
---

# Delegable

## Asset field

The extension is anchored on an asset address field. 

The extension defines a transaction with a `called by` test on the extended address value. This means that the extension will add the 2 actions \(`setDelegate` and `rmDelegate`\) for each transaction with a `called by` instruction on a matched address field.

```ocaml
archetype extension delegable (

  asset anAsset identified by aRole = {
    aRole[%delegable] : role
  }

  action aTransaction (anasset : anAsset) = {
     called by anasset.aRole
  }

) = {

  asset <%aRole>Operators identified by arole = {
    arole      : role;
    operators  : address set;
  }

  action setDelegate<%aRole> (arole : role) (operator : address) = {
    called by arole
    effect {
      (<%aRole>Operators.get owner).operators.add operator
    }
  }

  action rmDelegate<%aRole> (afield : role) (operator : address) = {
    called by arole
    effect {
      (<%aRole>Operators.get owner).operators.remove operator
    }
  }

  action aTransaction (anasset : anAsset) = {
    called by none
    require {
      (<%aRole>Operators.get anasset.arole).operators.contains caller
    }
  }
}
```

## Action

The extension is anchored on a called-by role. 

{% code-tabs %}
{% code-tabs-item title="action\_delelagable.arlx" %}
```ocaml
archetype extension actiondelegation (

  action aTransaction (anasset : anAsset) = {
    called by [%delegable] anasset.arole
  }

) = {

  action aTransaction = {
    effect {
      effects;
      <%aTransaction>_space::Operator.remove
      (<%aTransaction>_space::Operator.get anasset)
    }
  }

  namespace <%aTransaction>_space {
    asset Operator identified by anasset = {
      anasset  : anAsset;
      operator : address
    }

    action setDelegate (Operator : Operator) = {
      called by Operator.anasset.arole
      effect {
        Operator.add Operator
      }
    }

    action rmDelegate (Operator : Operator) = {
      called by Operator.anasset.arole

      effect {
        Operator.remove Operator
      }
    }

    action from_addr (anasset : anAsset) = {
      require {
        c1 : (Operator.get anasset).operator = caller
      }

      effect {
        Operator.remove (Operator.get anasset)
      }
    }

    specification = {
      (* assert that after token transfer no one 
                  is a delegate of this token … *)

    }
  }

}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

