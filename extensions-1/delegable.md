---
description: Delegation mechanism found in the ERC720 contract.
---

# Delegable

## Asset field

The extension is anchored on an asset address field. 

The extension defines a transaction with a `called by` test on the extended address value. This means that the extension will add the 2 actions \(`setDelegate` and `rmDelegate`\) for each transaction with a `called by` instruction on a matched address field.

{% code title="delegable.arlx" %}
```ocaml
archetype extension delegable (

  asset anAsset identified by aRole = {
    aRole[%delegable%] : role
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
      r1 : (<%aRole>Operators.get anasset.arole).operators.contains caller
    }
  }
}
```
{% endcode %}

## Action

The extension is anchored on a called-by role. 

{% code title="action\_delelagable.arlx" %}
```ocaml
archetype extension action_delegation (

  action aTransaction (anasset : anAsset) = {
    called by [%delegable%] anAsset.arole
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
  }
}


```
{% endcode %}

