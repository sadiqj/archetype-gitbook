---
description: The mutable extension adds a setter action for the extended variable.
---

# Mutable

This extension takes two arguments:

1. the role expression to specify who can change the value
2. the condition to change the value

{% embed url="https://gist.github.com/rognierbenoit/d6a61c1e7b2ec02f8b24" %}



```text
archetype extension mutable (

  variable[%mutable arole cond] avariable type

) = {

  action set_<%avariable> (new<%avariable> : type) = {
    called by arole
    require {
      set_<%avariable>_c1 : cond
    }
    effect {
      avariable := newvariable
    }
  }
}
```



