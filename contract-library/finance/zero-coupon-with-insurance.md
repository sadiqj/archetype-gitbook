# Zero coupon with insurance

The issue is that the previous zero coupon bond is that the owner has zero guarantee for repay.

The solution is to use a guarantee fund to ensure pay back. This fund plays the role of trusted third party for the zero coupon bond contract.

{% code-tabs %}
{% code-tabs-item title="zcb\_with\_insurance.arl" %}
```ocaml
archetype zero_coupon_bond_with_insurance

variable issuer role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk 
                       (* seller ‘Alice’ *)

variable owner role  = @tz1KmuhR6P52hw6xs5P69BXJYAURaznhvN1k
                       (* buyer ‘Bob’; receives 11 tez in one-year *)

contract insurance = {
   action pay : role, tez;
} = @KT1GabhR5P52hw6xs5P69BXJYAURaznhvN1k

value price tez from owner = 10

value payment tez from issuer to owner = 1.1 * price

value maturity date = now

states =
  | Created initial 
  | Insured         (* Guarantee Fund has accepted issuer *)
  | Confirmed       (* owner has purchased bond *)
  | Collected       (* owner has collected payment *)
  | Defaulted       (* at maturity, issuer hasn’t transferred payment *)


transition confirm from Created = {

  to Confirmed when { transferred = price }
  with effect {
    maturity := now + 1Y;
    transfer price to issuer
  }
}

transition insured from Created = {
  called by insurance
  to Insured
}

transition collect from Confirmed = {
  called by owner

  to Collected when { now >= maturity }
  with effect {
    if balance >= payment
    then transfer balance to owner
    else guarantee_fund.pay owner payment
  }

  to Defaulted when { now >= maturity and balance < payment }
}

transition confirm from Insured = {
  specification {
    balance = 0
  }

  to Confirmed when { transferred = price }
  with effect {
    maturity := now + 1Y;
    transfer price to issuer
  }
}

transition repay from Confirmed = {
  called by issuer

  to Repaid
  when { transferred = payment }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}



