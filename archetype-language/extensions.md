---
description: Factorise and reuse contract behaviour.
---

# Extension

## Introduction

Contracts, even simple ones, offer a large number of possible variations :

* may or may it not be possible to change the value of a parameter ?
* if so, should it be signed ? By how many authorised roles ?
* if a key role may be changed, what is the protocol ?
* …

Archetype provides the possibility to develop extensions in order to factorise these kind of processes and reuse them in several contracts.

An extension may be invoked at specific places in the code :

* declaration keywords \(archetype, variable, action, transition, contract, verification, enum, states, asset, effect, function\)
* field declaration \(of actions, transactions and functions\)
* action and transition sections \(called by, condition, effect, when, with effect\)

You may refer to the [archetype grammar](https://github.com/edukera/archetype-lang/blob/master/src/parser.mly) for the exact invocation location \(search for `exts=`\).

For example, the following invokes the ‘mutable’ extension on the amount variable:

```text
variable[%mutable admin] amount tez = 10tz
```

The mutable extension, presented in next section, adds to the contract an action to change the amount value. Once the extension processed, the declaration is extended to the following code:

```text
variable amount tez = 10tz

action set_amount (new_amount : tez) = {
  called by admin
  effect {
    amount := new_amount
  }
}
```

Extensions processing is a contract **pre-processing** and produces a new \(extended\) version of the contract.

## Rational

Why not use a more sophisticated code factorisation mechanism, like object inheritance or polymorphic typing mechanism ?  

More sophisticated mechanisms have the drawback to locate and hide contract code in other source files. Smart contracts need to be **fully transparent for security reasons**. 

The extension preprocessing produces a **single source** contract with all the executable behaviour.

## Create your own extension

The skeleton of an extension is as follows:

```text
archetype extension extensionName (

  (* code pattern *)

) = {

  (* extension body *)

}
```

The code pattern defines the matching code to extend. The extension body defines the code to add and/or transform.

An extension may call another extension, hence the preprocessor detects cyclic calls and emits an explicit error when detected. See the the [Signed by all](../extensions-1/signed-by-all.md) extension for an example.

## Extensions library

Archetype comes with a list of useful extensions:

{% page-ref page="../extensions-1/" %}

You may adapt them at your convenience.  


