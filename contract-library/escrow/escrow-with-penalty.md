# With penalty

The escrow archetype presented below defines an escrow process with a penalty for the seller if the transaction fails \(for example if it does not happen before the deadline\).

Factoring is enabled: the seller can sell the “invoice” to a creditor, and the buyer can sell the penalty to a debtor.

A state machine is used to follow the different stages of the escrow transaction.

{% code title="escrow\_penalty.arl" %}
```ocaml
archetype escrow_penalty

variable buyer : role = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

variable debitor : role = buyer

variable seller : role = @tz1bfVgcJC4ukaQSHUe1EbrUd5SekXeP9CWk

variable creditor : role = seller

variable oracle : role = @tz1iawHeddgggn6P5r5jtq2wDRqcJVksGVSa

variable price : tez = 10tz

variable penalty : tez = 0.1 * price

variable deadline : date = 2020-06-28T00:00:00

(* state machine *)
states =
 | Created initial
 | Aborted
 | Confirmed
 | Canceled
 | Transferred with { i1 : balance = 0tz; }

transition abort () {
  called by buyer or seller

  from Created
  to Aborted
}

transition confirm () {
  from Created
  to Confirmed when { balance = price + penalty }
}

transition transfer_ () {
  called by oracle

  from Confirmed
  to Transferred when { now < deadline }
  with effect {
    transfer price to creditor;
    transfer penalty to seller
  }
}

transition cancel () {
  called by oracle

  from Confirmed
  to Canceled
  with effect {
    transfer penalty to debitor;
    transfer price to buyer
  }
}

```
{% endcode %}



