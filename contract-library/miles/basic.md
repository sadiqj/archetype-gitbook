# Basic

This archetype is a very basic miles management system with add and consume transactions. It may be considered as a simplified version of ERC20.

{% code title="miles.arl" %}
```ocaml
archetype miles

variable owner role

variable[%transferable%] validator role

asset account identified by owner = {
  owner  : address;
  amount : uint
}

action add (owner : address) (value : uint) = {
  called by validator
  effect {
    account.update owner { amount += value } { amount = 0 }
  }
}

action consume (owner : address) (value : uint) = {
  called by validator

  require {
    c1 : (account.get owner).amount >= value
  }


  specification {
    s1 : (before.account.get owner).amount =
           (after.account.get owner).amount + value
  }

  effect {
    account.update owner {amount -= value}
  }
}

```
{% endcode %}

