# Formal properties

See presentation article:

{% embed url="https://assets.tqtezos.com/docs/token-contracts/fa12/4-fa12-archetype/" %}

## Declarations

The contract declares two assets:

### **ledger asset**

```coffeescript
asset ledger identified by holder to big_map {
  holder     : address;
  tokens     : nat = 0;
} initialized by {
  { holder = caller; tokens = totalsupply }
}
```

The `ledger` asset is the cap table: it holds the number of tokens for each token holder. `totalsupply` is the initial number of tokens held by the originator of the contract.

### **allowance asset**

```coffeescript
asset allowance identified by addr_owner addr_spender to big_map {
  addr_owner       : address;
  addr_spender     : address;
  amount           : nat;
}
```

The `allowance` asset stores the amount of tokens that can be spent by `addr_spender` on the behalf of `addr_owner`.

## Contract invariant

```javascript
ledger.sum(tokens) = totalsupply
```

> No token is minted: the total number of tokens is equal to the initial `totalsupply` number of tokens, regardless of the state of the storage.

## transfer

Archetype implementation:

```coffeescript
entry %transfer (%from : address, %to : address, value : nat) {
  require {
    r1 otherwise "NotEnoughBalance" : ledger[%from].tokens >= value;
  }
  effect {
    if caller <> %from then (
      var current = allowance[(%from, caller)].amount;
      dofailif(current < value, ("NotEnoughAllowance", ((value, current))));
      allowance.update((%from, caller), { amount -=  value });
    );
    ledger.update(%from, { tokens -= value });
    ledger.addupdate(%to, { tokens += value });
  }
}
```

### **1/ Effect on ledger**

#### **1.1/ Effect on %from token holder**

```ocaml
%from <> %to ->
let some before_ledger_from = before.ledger[%from] in
let some after_ledger_from  = ledger[%from] in
  after_ledger_from = { before_ledger_from with
    tokens = (before_ledger_from.tokens - value)
  }
otherwise false
otherwise false
```

> When the `%to` address is different from the `%from` address, the number of tokens `%to` possesses is decread by `value`.

#### **1.2/ Effect on %to token holder**

```ocaml
%from <> %to ->
let some after_ledger_to = ledger[%to] in
let some before_ledger_to = before.ledger[%to] in
  after_ledger_to = { before_ledger_to with
    tokens = (before_ledger_to.tokens + value)
  }
otherwise
  after_ledger_to = { holder = %to; tokens = value }
otherwise false
```

> When the `%to` address is different from the `%from` address, the number of tokens `%to` possesses is increased by `value` when `%to` is already registered in the ledger, and set to `value` otherwise. Anyway, `%to` is registered in ledger.

#### **1.3/ No effect on ledger**

```ocaml
%from = %to -> ledger = before.ledger
```

> No effect on ledger when `%from` is equal to `%to`.

#### **1.4/ Unchanged ledger records**

```ocaml
forall tokenholder in ledger,
tokenholder.holder <> %from ->
tokenholder.holder <> %to ->
before.ledger[tokenholder.holder] = some(tokenholder)
```

> Tokenholders other than `%from` and `%to`, are not modified nor added to `ledger`.

#### **1.5/ Removed ledger records**

```ocaml
removed.ledger.isempty()
```

> No record is removed from `ledger`.

#### **1.6/ Added ledger records**

```ocaml
let some before_to = before.ledger[%to] in
  added.ledger.isempty()
otherwise
  added.ledger = [ { holder = %to; tokens = value } ]
```

> The added record in 'ledger', if any, is the `%to` record.

### **2/ Effect on allowance**

#### **2.1/ Effect on \(%from,caller\) allowance record**

```ocaml
caller <> %from ->
let some before_from_caller = before.allowance[(%from,caller)] in
let some after_from_caller = allowance[(%from,caller)] in
  before_from_caller.amount > value ->
  after_from_caller = { before_from_caller with
    amount = (before_from_caller.amount - value)
  }
otherwise false
otherwise true
```

> When `caller` is different from `%from`, the amount `caller` is authorised to spend on the behalf of `%from` is decreased by `value` if `value` is striclty greated than the authorized amount.

#### **2.1/ No effect on allowance**

```ocaml
caller = %from -> allowance = before.allowance
```

> No effect on `allowance` when `caller` is equal to `%from`.

#### **2.2/ Unchanged allowance records**

```ocaml
forall a in allowance,
a.addr_owner <> %from and a.addr_spender <> caller ->
before.allowance[(a.addr_owner,a.addr_spender)] = some(a)
```

> Allowed amounts other than those associated to `%from` and `caller` are identical.

#### **2.3/ Added and removed allowance records**

```javascript
removed.allowance.isempty() and added.allowance.isempty()
```

> No allowance record is added or removed.

### **3/ Explicit fail**

#### **3.1/ Not Enough Balance**

```ocaml
fails with (msg : string) :
  let some after_ledger_from = ledger[%from] in
    msg = "NotEnoughBalance" and
    after_ledger_from.tokens < value
  otherwise true
```

> When the entry fails with message "NotEnoughBalance", `value` is stricly greater than the number of tokens of `%to`. Cannot spend more than you own.

#### **3.2/ Not Enough Allowance**

```javascript
fails with (msg : string * (nat * nat)) :
  let some after_allowance_from_caller = allowance[(%from,caller)] in
    msg = ("NotEnoughAllowance", (value, after_allowance_from_caller.amount)) and
    caller <> %from and
    after_allowance_from_caller.amount < value
  otherwise false
```

> When the entry fails with message "NotEnoughAllowance", `caller` is different from `%from` and `value` is stricly greater than the allowed amount for `%from` and `caller`. A spender cannot spend more than allowed.

### **4/ Effect on operations**

```javascript
length (operations) = 0
```

> No operation generated.

## approve

Archetype implementation:

```coffeescript
entry approve(spender : address, value : nat) {
  var k = (caller, spender);
  if allowance.contains(k) then (
    var previous = allowance[k].amount;
    dofailif(previous > 0 and value > 0, (("UnsafeAllowanceChange", previous)));
  );
  allowance.addupdate( k, { amount = value });
}
```

### **1/ Effect on ledger**

```ocaml
ledger = before.ledger
```

> No effect on `ledger`.

### **2/ Effect on allowance**

#### **2.1/ Effect on \(caller,spender\) allowance record**

```ocaml
let some after_allowance_caller_spender = allowance[(caller,spender)] in
let some before_allowance_caller_spender = before.allowance[(caller,spender)] in
  after_allowance_caller_spender = { before_allowance_caller_spender with
    amount = value
  }
otherwise
  after_allowance_caller_spender = {
    addr_owner = caller;
    addr_spender = spender;
    amount = value
  }
otherwise false
```

> Allowed amount of tokens spendable by `spender` on the behalf of `caller` is set to `value`.

#### **2.2/ Unchanged allowance records**

```ocaml
forall a in allowance,
(a.addr_owner, a.addr_spender) <> (caller, spender) ->
before.allowance[(a.addr_owner, a.addr_spender)] = some(a)
```

> Other allowed amounts than the allowed amount of tokens spendable by `spender` on the behalf of `caller`, are unchanged.

#### **2.3/ Added allowance records**

```javascript
let some allowance_caller_spender = before.allowance[(caller, spender)] in
  added.allowance.isempty()
otherwise
  added.allowance = [ { addr_owner = caller; addr_spender = spender; amount = value } ]
```

> The added `allowance` record, if any, is the `caller` and `spender` one.

#### **2.4/ Removed allowance records**

```javascript
removed.allowance.isempty()
```

> No record is removed from `allowance`.

#### **3/ Explicit fail**

```ocaml
fails with (msg : (string * nat)) :
let some allowance_caller_spender = allowance[(caller,spender)] in
  msg = ("UnsafeAllowanceChange", allowance_caller_spender.amount) and
  value > 0 and
  allowance_caller_spender.amount > 0
otherwise false
```

> When the entry fails with message "UnsafeAllowanceChange", `value` is strictly greater than 0 and the allowed amount of tokens spendable by `spender` on the behalf of `caller` is not equal to zero.

Note that in that case, it may be set back to 0 by having `spender` call the `transfer` entry to transfer a number of tokens equal to the remaining allowed amount, from the approver address \(ie `caller` above\) to the approver address \(ie to itself\).

Indeed, according to properties 1.3 and 2.1 of the `transfer` entry, this has no effect on `ledger` and sets allowance record to 0.

#### **4/ Effect on operations**

```javascript
length (operations) = 0
```

> No operation generated.

## getAllowance

Archetype implementation:

```objectivec
getter getAllowance (owner : address, spender : address) : nat {
  return (allowance[(owner, spender)].amount)
}
```

### **1/ Effect on ledger**

```javascript
ledger = before.ledger
```

> No effect on `ledger`.

### **2/ Effect on allowance**

```javascript
allowance = before.allowance
```

> No effect on `allowance`.

### **3/ Explicit fail**

No explicit fail. The entry implicitely fails though if the provided callback is invalid.

### **4/ Effect on operations**

```javascript
length (operations) = 1
```

> Creates one callback operation.

## getBalance

Archetype implementation:

```coffeescript
getter getBalance (owner : address) : nat {
  return (ledger[owner].tokens)
}
```

### **1/ Effect on ledger**

```javascript
ledger = before.ledger
```

> No effect on `ledger`.

### **2/ Effect on allowance**

```javascript
allowance = before.allowance
```

> No effect on `allowance`.

### **3/ Explicit fail**

No explicit fail. The entry implicitely fails though if the provided callback is invalid.

### **4/ Effect on operations**

```javascript
length (operations) = 1
```

> Creates one callback operation.

## getTotalSupply

Archetype implementation:

```coffeescript
getter getTotalSupply () : nat {
  return totalsupply
}
```

### **1/ Effect on ledger**

```ocaml
ledger = before.ledger
```

> No effect on `ledger`.

### **2/ Effect on allowance**

```javascript
allowance = before.allowance
```

> No effect on `allowance`.

### **3/ Explicit fail**

No explicit fail. The entry implicitely fails though if the provided callback is invalid.

### **4/ Effect on operations**

```javascript
length (operations) = 1
```

> Creates one callback operation.

