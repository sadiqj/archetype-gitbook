# Zero coupon with insurance

The issue is that the previous zero coupon bond is that the owner has zero guarantee for repay.

The solution is to use a guarantee fund to ensure pay back. This fund plays the role of trusted third party for the zero coupon bond contract.

{% tabs %}
{% tab title="zcb\_with\_insurance.arl" %}
```ocaml
archetype zero_coupon_bond_with_insurance

variable issuer role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk (* seller ‘Alice’ *)

variable owner role  = @tz1KmuhR6P52hw6xs5P69BXJYAURaznhvN1k
(* buyer ‘Bob’; receives 11 tez in one-year *)

contract insurance = {
   action pay : address, tez
} 

variable zero_insur insurance = @KT1GabhR5P52hw6xs5P69BXJYAURaznhvN1k

variable price tez from owner = 10

variable payment tez from issuer to owner = 11 * price

variable maturity date

states =
 | Created initial
 | Insured   (* Guarantee Fund has accepted issuer *)
 | Confirmed (* owner has purchased bond *)
 | Repaid    (* issuer has transferred payment to contract *)
 | Collected (* owner has collected payment *)

transition insured from Created = {
  called by zero_insur

  to Insured
}

transition confirm from Insured = {
  specification {
    s1 : balance = 0
  }

  to Confirmed
  when { transferred = price }
  with effect {
    maturity := now + 1Y;
    transfer price to issuer
  }
}

transition repay from Confirmed = {
  called by issuer

  to Repaid
  when {
    transferred = payment
  }
}

transition collect from Repaid = {
  called by owner

  to Collected
  when { now > maturity }
  with effect {
    if balance >= payment
    then transfer balance to owner
    else zero_insur.pay owner payment
  }
}

```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="guarantee\_fund.arl" %}
```ocaml
archetype guarantee_fund

variable admin role

asset insured_contract identified by addr = {
  addr : address;
  max_transfer : tez
}

action credit = {
  called by admin
  require {
    c1 : transferred > 0
  }
}

action add_contract (contract_ : insured_contract) = {
  called by admin
  effect {
    insured_contract.addifnotexist contract_
  }
}

action pay (recipient : address) (amount : tez) = {
  require {
    c1 : insured_contract.contains caller;
    c2 : amount <= (insured_contract.get caller).max_transfer
  }

  effect {
    transfer amount to recipient
  }
}
```
{% endtab %}
{% endtabs %}

