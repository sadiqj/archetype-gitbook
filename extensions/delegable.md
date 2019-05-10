---
description: >-
  This extension factorises the delegation mechanism found in the ERC721
  contract.
---

# Delegable

The extension is anchored on an asset address field. 

The extension defines a transaction with a `called by` test on the extended address value. This means that the extension will add the 2 actions \(`setDelegate` and `rmDelegate`\) for each transaction with a `called by` instruction on a matched address field.

```text
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

