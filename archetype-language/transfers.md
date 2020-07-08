# Transfers, contract calls

## Transfer

The `transfer` instruction is used to transfer Tezis to an account.

```javascript
effect {
  transfer 1tz to coder;
}
```

## Calling a contract

It is possible to call a smart contract with the `transfer` instruction above.

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

* declare the contract signature with the `contract` keyword
* declare a variable typed with this signature and give it the address value of the called contract 

```javascript
contract contract_called_sig {
   entry set_value (n : int)
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

