# With penalty

The escrow archetype presented below defines an escrow process with a penalty for the seller if the transaction fails \(for example if it does not happen before the deadline\).

Factoring is enabled: the seller can sell the “invoice” to a creditor, and the buyer can sell the penalty to a debtor.

A state machine is used to follow the different stages of the escrow transaction.

{% code title="escrow\_penalty.arl" %}
```ocaml
archetype escrow_penalty

variable buyer : role = @tz1buyer

variable[%transferable%] debitor : role = buyer

variable seller : role = @tz1seller

variable[%transferable%] creditor : role = seller

variable oracle : role = @tz1oracle

variable[%traceable%] price : tez = 10tz

variable[%traceable%] [%mutable_signed ([{buyer}; {debitor}], (state = Created))%]
     penalty : tez = 0.1 * price

(* action deadline *)
variable[%mutable ((buyer or seller), (instate (Created)))%] deadline : date = 2020-06-28T00:00:00

(* state machine *)
states =
 | Created initial
 | Aborted
 | Confirmed
 | Canceled
 | Transferred with { i1 : balance = 0tz; }

transition abort () from Created {
  called by buyer or seller

  to Aborted
}

transition[%signedbyall [buyer; seller]%] confirm () from Created {
  to Confirmed when { balance = price + penalty }
}

transition transfer_ () from Confirmed {
  called by oracle

  to Transferred when { now < deadline }
  with effect {
    transfer price to creditor;
    transfer penalty to seller
  }
}

transition cancel () from Confirmed {
  called by oracle

  to Canceled
  with effect {
    transfer penalty to debitor;
    transfer price to buyer
  }
}

```
{% endcode %}



