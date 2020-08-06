# Zero coupon with insurance

The issue is that the previous zero coupon bond is that the owner has zero guarantee for repay.

The solution is to use a guarantee fund to ensure pay back. This fund plays the role of trusted third party for the zero coupon bond contract.

{% code title="zcb\_with\_insurance.arl" %}
```ocaml
archetype zero_coupon_bond_with_insurance

variable issuer : role = @tz1bfVgcJC4ukaQSHUe1EbrUd5SekXeP9CWk (* seller 'Alice' *)

variable owner : role  = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg
(* buyer 'Bob'; receives 11 tez in one-year *)

variable zero_insur : address = @KT1TJrR7uovV5tFpsLBCKmx3x95pY6hMy775

variable price : tez = 10tz

variable payment : tez = 1.1 * price

variable maturity : date = now

states =
 | Created initial
 | Insured   (* Guarantee Fund has accepted issuer *)
 | Confirmed (* owner has purchased bond *)
 | Repaid    (* issuer has transferred payment to contract *)
 | Collected (* owner has collected payment *)

transition insured () {
  called by zero_insur

  from Created
  to Insured
}

transition confirm () {
  specification {
    s1 : balance = 0tz
  }

  from Insured
  to Confirmed
  when { transferred = price }
  with effect {
    maturity := now + 365d;
    transfer price to issuer
  }
}

transition repay () {
  called by issuer

  from Confirmed
  to Repaid
  when {
    transferred = payment
  }
}

transition collect () {
  called by owner

  from Repaid
  to Collected
  when { now > maturity }
  with effect {
    if balance >= payment
    then transfer balance to owner
    else transfer 0tz to zero_insur call pay<address * tez>((owner, payment))
  }
}

```
{% endcode %}

{% code title="guarantee\_fund.arl" %}
```ocaml
archetype guarantee_fund

variable admin : role = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

asset insured_contract identified by addr {
  addr : address;
  max_transfer : tez;
}

entry credit () {
  called by admin
  require {
    c1 : transferred > 0tz;
  }
}

entry add_contract (ic_addr : address, ic_max_transfer : tez) {
  called by admin
  effect {
    insured_contract.add({addr = ic_addr; max_transfer = ic_max_transfer})
  }
}

entry pay (recipient : address, amount : tez) {
  require {
    c2 : insured_contract.contains(caller);
    c3 : amount <= insured_contract[caller].max_transfer;
  }

  effect {
    transfer amount to recipient
  }
}

```
{% endcode %}

