# With penalty

The escrow archetype presented below defines an escrow process with a penalty for the seller if the transaction fails \(for example if it does not happen before the deadline\).

Factoring is enabled: the seller can sell the “invoice” to a creditor, and the buyer can sell the penalty to a debtor.

A state machine is used to follow the different stages of the escrow transaction.

{% code-tabs %}
{% code-tabs-item title="escrow\_penalty.arl" %}
```ocaml
archetype escrow

variable buyer role

variable[%transferable] debitor role = buyer

variable seller role

variable[%transferable] creditor role = seller

variable oracle role

variable[%traceable] price tez from buyer to creditor

variable[%traceable] [%mutable_signed [buyer, debitor] (state = Created)]
     penalty tez from seller to debitor = 0.1 * price

(* action deadline *)
variable[%mutable (buyer or seller) (instate Created)] deadline date

(* state machine *)
states =
 | Created initial
 | Aborted
 | Confirmed
 | Canceled
 | Transferred with { i1 : balance = 0}

transition abort from Created = {
  called by buyer or seller

  to Aborted
}

transition[%signedbyall [{buyer}; {seller}]] confirm from Created = {
  to Confirmed when balance = price + penalty
}

transition transfer_ from Confirmed = {
  called by oracle

  to Transferred when now < deadline
  with effect {
    transfer price;
    transfer back penalty
  }
}

transition cancel from Confirmed = {
  called by oracle

  to Canceled
  with effect {
    transfer penalty;
    transfer back price
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}



