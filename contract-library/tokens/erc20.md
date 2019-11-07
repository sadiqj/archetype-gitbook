---
description: The one and only
---

# ERC20

The standard may be found at this address:

[https://theethereum.wiki/w/index.php/ERC20\_Token\_Standard](https://theethereum.wiki/w/index.php/ERC20_Token_Standard)

{% tabs %}
{% tab title="erc20.arl" %}
```ocaml
archetype erc20

constant name string = "myerc20"

constant total uint = 1000

asset tokenHolder identified by holder = {
    holder : role;
    tokens : int;
} with {
    i1: tokens >= 0;
} initialized by [
  { holder = caller; tokens = total }
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

security {
   s1 : only_in ();
}
```
{% endtab %}
{% endtabs %}

