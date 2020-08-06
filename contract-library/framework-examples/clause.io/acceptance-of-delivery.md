# Acceptance of delivery

> Acceptance of Delivery. shipper will be deemed to have completed its delivery obligations if in receiver's opinion, the deliverable satisfies the Acceptance Criteria, and receiver notifies shipper in writing that it is accepting the deliverable.
>
> Inspection and Notice. receiver will have businessDays Business Days' to inspect and evaluate the deliverable on the delivery date before notifying shipper that it is either accepting or rejecting the deliverable.
>
> Acceptance Criteria. The "Acceptance Criteria" are the specifications the deliverable must meet for the shipper to comply with its requirements and obligations under this agreement, detailed in attachment, attached to this agreement.

{% embed url="https://github.com/clauseHQ/cicero-template-library/tree/master/src/acceptance-of-delivery" %}

```ocaml
archetype clause_io_acceptance_of_delivery

variable shipper : role = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

variable receiver : role = @tz1bfVgcJC4ukaQSHUe1EbrUd5SekXeP9CWk

variable payment : tez = 10tz

variable deliveryDate : date = 2020-12-31

variable businessDays : duration = 14d

(* extra blockchain incentives *)
variable incentiveR : tez = payment

variable incentiveS : tez = 0.2 * payment

(* %traceable (payment + incentiveR, receiver) *)
(* %traceable (incentiveS, shipper) *)

states =
 | Created initial
 | Aborted
 | Signed
 | Delivered
 | Success
 | Fail

transition sign () {
  from Created
  to Signed
  when { balance = payment + incentiveR + incentiveS }
}

transition unilateral_abort () {
  called by shipper or receiver

  from Created
  to Aborted
  with effect {
    transfer incentiveR to receiver;
    transfer payment to receiver;
    transfer incentiveS to receiver
  }
}

transition abort () {
  called by shipper or receiver

  from Signed
  to Aborted
  with effect {
    transfer incentiveR to receiver;
    transfer payment to receiver;
    transfer incentiveS to receiver
  }
}

transition confirm () {
  called by receiver

  from Signed
  to Delivered
  with effect {
     deliveryDate := now;
     transfer incentiveR to receiver;
     transfer payment to shipper;
     transfer incentiveS to receiver
  }
}

transition success () {
  called by shipper

  from Delivered
  to Success
  when { now > (deliveryDate + businessDays) }
}

transition fail_ () {
  called by receiver

  from Delivered
  to Fail
  when { now <= deliveryDate + businessDays }
}

```

