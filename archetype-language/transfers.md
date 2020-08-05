# Transfers, contract calls

## Transfer

The `transfer` instruction is used to transfer Tezis to an account.

```javascript
effect {
  transfer 1tz to coder;
}
```

where coder is a `role` variable.

## Calling a contract

It is possible to call a smart contract with the `transfer` instruction. Say for example you want to call the `set_value` entry point of the following contract: 

```javascript
archetype contract_called

variable v : nat = 0

entry set_value(x : nat) { v := x }
```

The contract may then be called with the `transfer ... to ... call ...` instruction:

```javascript
effect {
    var c = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm;
    transfer 0tz to c call add_value<nat>(3);
}
```

### Self

It is possible to call the current contract itself. Say the current contract has the `my_add_value` entry point; it is possible to call it from another entry point with the following instruction:

```javascript
entry my_add_value(int a, int b) {
   sum := a + b;
}

entry add_1_3 () {
   transfer 0tz to entry self.my_add_value(1,3)
}
```

### Callback

A common pattern for contracts to exchange values is to call an entry point with a callback.

This pattern is presented in the _two-way-inter-contract_ _invocation_ article below:

{% embed url="https://medium.com/protofire-blog/enabling-smart-contract-interaction-in-tezos-with-ligo-functions-and-cps-e3ea2aa49336" %}

The archetype version of the _sender_ contract \(see article above\):

```javascript
archetype sender

variable bar : int = 0

entry getBar (cb : entrysig<int>) { transfer 0tz to entry cb(bar) }

entry setBar (b : int) { bar := b }
```

The archetype version of the _inspector_ contract \(see article above\):

```javascript
archetype inspector

variable foo : int = 0

entry setFoo(v : int) { foo := v }

entry getFoo(asender : address) { 
  transfert 0tz to asender call getBar<entrysig<int>>(self.setFoo) 
}
```

