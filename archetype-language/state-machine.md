---
description: Contract as a state machine (states and transitions)
---

# State machine

When possible, defining a contract as a state machine is very convenient because it makes it simpler to:

* get an insight in the global contract mechanism \(easier to read and understand\)
* formally verify the contract properties \(more assumptions can be done on a state\) 

## States

It is possible in Archetype to define a global contract state . The following example creates 5 states:

```ocaml
states = 
 | Created initial
 | Confirmed
 | Completed
 | Aborted
 | Canceled
```

`initial` is a keyword to set the default state.

When necessary, action effects may read the state value \(but cannot change it\), as in the example below:

```ocaml
effect {
  ...
  match state with
  | Created -> fail ("cannot do that in Created state")
  end;  
  ...
}
```

## Transitions

Transitions are necessary to change the state of the contract. For example the following snippet declares the transition `confirm` to go from `Created` to `Confirmed`:

```ocaml
transition confirm () {
  from Created
  to Confirmed 
  with effect {
    ...          (* after this, state is set to Created *)
  }
}
```

Like actions, transitions may have arguments, and also have the `called by` `specification` and `require`  sections.

It is also possible to decide the resulting state. For example the following `decide` transition determine the resulting state based on a condition `<COND>` :

```ocaml
transition mayconfirm () {
  from Created
  to Confirmed when {<COND>}
  with effect {
     ...       (* after this, state is set to Created *)
  }
  
  to Canceled  (* execute this when <COND> is false *)
  with effect {
     ...       (* after this, state is set to Canceled *)
  } 
}
```

## Example

Follow the link below to observe the basic escrow example implemented as a state machine.

{% page-ref page="../contract-library/escrow/basic-escrow.md" %}

