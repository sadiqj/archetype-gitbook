---
description: The one and only
---

# ERC20

The standard may be found at this address:

{% embed url="https://theethereum.wiki/w/index.php/ERC20\_Token\_Standard" %}

{% tabs %}
{% tab title="erc20.arl" %}
```ocaml
archetype erc20

constant name string = "myerc20"

constant total uint = 1000

asset allowance {
    allowed : address;
    amount  : int;
} with {
    i2 : amount > 0;
}

asset tokenHolder identified by holder = {
    holder     : address;
    tokens     : int;
    allowances : allowance partition
} with {
    i1: tokens >= 0;
    i2: allowances.sum(the.amount) <= tokens;
} initialized by [
  { holder = caller; tokens = total; allowances = [] }
]

action transfer (to : key of tokenHolder) (value : uint) = {

  specification {
    p1 : before tokenHolder.sum(tokens) = tokenHolder.sum(tokens);
    p2 : let th = tokenHolder.get(th) in
         th.tokens = before.th.tokens + value
         otherwise true
    p3 : let thc = tokenHolder.get(caller) in 
         thc.tokens = before.thc.tokens - value
         otherwise true
    p4 : forall t in tokenHolder,
         if t <> th then
         if t <> caller then
         t.tokens = before.t.tokens
  }

  failif {
    f1 : tokenHolder.get(caller).tokens < value
  }

  effect {
    tokenHolder.update( to.holder, { tokens += value });
    tokenHolder.update( caller, { tokens -= value })
  }
}
```
{% endtab %}
{% endtabs %}

