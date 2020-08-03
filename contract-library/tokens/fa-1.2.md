---
description: Fungible token for Tezos
---

# FA 1.2

FA 1.2 is the fungible token specification for Tezos :

{% embed url="https://assets.tqtezos.com/docs/token-contracts/fa12/1-fa12-intro/" %}

Below is the archetype implementation:

```javascript
archetype fa12

constant totalsupply : nat = 1000000000000000

asset allowance identified by addr_owner addr_spender to big_map {
  addr_owner       : address;
  addr_spender     : address;
  amount           : nat;
}

asset ledger identified by holder to big_map {
  holder     : address;
  tokens     : nat = 0;
} initialized by {
  { holder = caller; tokens = totalsupply }
}

entry %transfer (from_ : address, to_ : address, value : nat) {
  require {
    d0 : ledger[from_].tokens >= value
  }
  effect {
    if caller <> from_ then (
      dorequire (allowance[(from_,caller)].amount >= value);
      allowance.update((from_,caller), { amount -=  value });
    );
    ledger.update(from_, { tokens -= value });
    ledger.addupdate(to_, { tokens += value });
  }
}

entry approve(spender : address, value : nat) {
    allowance.addupdate((caller, spender), { amount = value });
}

entry getAllowance (owner : address, spender : address, cb : entrysig<nat>) {
  transfer 0tz to entry cb(allowance[(owner,spender)].amount);
}

entry getBalance (owner : address, cb : entrysig<nat>) {
  transfer 0tz to entry cb(ledger[owner].tokens);
}

entry getTotalSupply (cb : entrysig<nat>) {
  transfer 0tz to entry cb(totalsupply);
}
```

