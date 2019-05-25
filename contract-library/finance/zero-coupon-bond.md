# Zero coupon bond

This contract is the archetype version of the contract taken from the [Findel paper](http://orbilu.uni.lu/handle/10993/30975) \(Financial Derivatives Language\).

Suppose Alice sells to Bob a zero-coupon \(i.e., paying no interest\) bond that pays $11 in one year for $10.

The findel expression below describes this contract:

```text
And ( Give( Scale( 10; One( USD ))); At( now+1 years; Scale( 11 ; One( USD ))))
```

{% code-tabs %}
{% code-tabs-item title="zero\_coupon\_bond.arl" %}
```ocaml
archetype zero_coupon_bond

variable issuer role  = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk (* seller ‘Alice’ *)

variable owner role = @tz1KmuhR6P52hw6xs5P69BXJYAURaznhvN1k
(* buyer ‘Bob’; receives 11 tez in one-year *)


variable price tez from owner = 10

variable payment tez from issuer to owner = 11 * price

variable maturity date = now

states =
  | Created initial
  | Confirmed (* owner has purchased bond *)
  | Collected (* owner has collected payment *)
  | Defaulted (* at maturity, issuer hasn’t transferred payment *)

transition confirm from Created = {

  to Confirmed when { transferred = price }
  with effect {
    maturity := now + 1Y;
    transfer price to issuer
  }
}

transition terminate from Confirmed = {
  called by owner

  to Collected when { now >= maturity and balance >= payment }
  with effect {
    transfer balance to owner
  }

  to Defaulted when { now >= maturity and balance < payment }

}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

