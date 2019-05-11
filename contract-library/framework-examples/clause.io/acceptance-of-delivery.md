# Acceptance of delivery

> Acceptance of Delivery. shipper will be deemed to have completed its delivery obligations if in receiver's opinion, the deliverable satisfies the Acceptance Criteria, and receiver notifies shipper in writing that it is accepting the deliverable.
>
> Inspection and Notice. receiver will have businessDays Business Days' to inspect and evaluate the deliverable on the delivery date before notifying shipper that it is either accepting or rejecting the deliverable.
>
> Acceptance Criteria. The "Acceptance Criteria" are the specifications the deliverable must meet for the shipper to comply with its requirements and obligations under this agreement, detailed in attachment, attached to this agreement.

{% embed url="https://github.com/clauseHQ/cicero-template-library/tree/master/src/acceptance-of-delivery" %}

{% code-tabs %}
{% code-tabs-item title="acceptance\_of\_delivery.arl" %}
```ocaml
archetype clause_io_acceptance_of_delivery

variable shipper role

variable receiver role

variable payment tez from receiver to shipper

variable deliveryDate date

variable businessDays day

(* extra blockchain incentives *)
variable incentiveR tez from receiver = payment

variable incentiveS tez from receiver = 0.2 * payment

%traceable (payment + incentiveR) (receiver)
%traceable (incentiveS) (shipper)

states =
 | Created initial
 | Aborted
 | Signed
 | Delivered
 | Success
 | Fail

transition[%signedbyall [{shipper}; {receiver}]] sign from Created = {
  to Signed
  when { balance = payment + incentiveR + incentiveS }
}

transition unilateral_abort from Created = {
  called by shipper or receiver

  to Aborted
  with effect {
    transfer back incentiveR;
    transfer back payment;
    transfer back incentiveS
  }
}

transition[%signedbyall [{shipper}; {receiver}]] abort from Signed = {
  called by shipper or receiver

  to Aborted
  with effect {
    transfer back incentiveR;
    transfer back payment;
    transfer back incentiveS
  }
}

transition confirm from Signed = {
  called by receiver

  to Delivered
  with effect {
     deliveryDate := now;
     transfer back incentiveR;
     transfer payment;
     transfer back incentiveS
  }
}

transition success from Delivered = {
  called by shipper

  to Success
  when { now > deliverydate + businessDays }
}

transition fail_ from Delivered = {
  called by receiver

  to Fail
  when { now <= deliverydate + businessDays }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

