# Transfers, contract calls

## Transfer

The `transfer` instruction is used to transfer Tezis to an account.

```javascript
effect {
  transfer 1tz to coder;
}
```

## Calling a contract

It is possible to call a smart contract with the `transfer` instruction.

### Interface

Say for example you want to call the `add_value` entry of the following contract: 

```javascript
archetype contract_called

variable v : int = 0

entry add_value(a : int, b : int) {
  effect {
    v := a + b
  }
}
```

In the calling contract, you need to:

* declare the contract interface with the `contract` keyword
* declare a variable typed with this signature and give it the address value of the called contract 

```javascript
contract contract_called_sig {
   entry add_value (a : int, b : int)
}

variable c : contract_called_sig = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm
```

The contract may then be called with the `transfer` instruction:

```javascript
effect {
    transfer 0tz to c call add_value(3,2);
}
```

Note that the contract interface may _not_ list the entire target contract entry points. It may even declare non existing entries as long as they are not called, even if it is not recommended for parsimony reason.

### Entrypoint

It is possible to declare a single entry point to a contract. The type for an entry point is `entrysig` followed by its signature. Operator `entrypoint` builds an entry point with the contract address and the name of the entry point.

Say for example you want to call the `createtoken` entrypoint of the following contract:

```javascript
archetype token

constant admin : address = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

asset holder {
  id : address;
  nb : nat = 0;
}

entry createtoken(h : pkey<holder>, n : nat) {
   called by admin
   effect {
      holder.addupdate(h, { nb += n });
   }
}
```

Then you may call the `createtoken` entry with the following:

```javascript
effect {
   var ctopt : option<entrysig<address * nat>> = entrypoint(token,"createtoken");
   if issome(ct) then (
      var ct = getopt(ctopt);
      transfer 0tz to entry ct(anaddress,10)
   )
   else fail "no contract found or no entrypoint 'createtoken' found"
}
```



