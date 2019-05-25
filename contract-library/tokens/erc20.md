---
description: The one and only
---

# ERC20

The standard may be found at this address:

[https://theethereum.wiki/w/index.php/ERC20\_Token\_Standard](https://theethereum.wiki/w/index.php/ERC20_Token_Standard)

{% code-tabs %}
{% code-tabs-item title="erc20.arl" %}
```ocaml
archetype erc20

constant name string = "myerc20"

constant total uint = 1000

asset tokenHolder identified by holder = {
    holder[%delegable%] : role;
    tokens : uint;
} initialized by [
  { caller; total }
]

action transferTokens (th : tokenHolder) (quantity : uint) = {
  called by th.holder

  specification {
    s1 : before tokenHolder.sum(tokens) = tokenHolder.sum(tokens)
  }

  effect {
    tokenHolder.update th.holder {tokens += quantity};
    tokenHolder.update caller {tokens -= quantity}
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

