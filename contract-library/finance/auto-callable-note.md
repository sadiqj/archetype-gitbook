# Auto-callable note

This contract is the transcription to archetype of the following auto-callable note from Goldman Sachs:

[https://www.eavest.com/products/e1f62344-818a-39a1-a77c-75ad849a07d6/tsheet](https://www.eavest.com/products/e1f62344-818a-39a1-a77c-75ad849a07d6/tsheet)

Thank you to Alain Frisch, CTO at [Lexifi](https://www.lexifi.com/), for this example contract and his help to transcode it. Lexifi provides financial services with a DSL \(Domain Specific Language\) to execute and simulate financial contracts. Alain Frish also provided the corresponding Lexifi code, automatically extracted by Lexifi drivers from the above document.

The key here is that a smart contract is passive regarding decisions to transfer currencies: a smart contract cannot be programmed to transfer or withdraw currencies : a transfer of currency results from a call to the contract, and there is no instruction for a contract to withdraw currencies from another account.

As a result, the contract is designed as follows: the issuer transfers to the contract, interests and redeem amount in due time \(the contract serves as an escrow account\); the owner can withdraw transferred amount at any time and can check, at any time, the total transferred, if he thinks there is a default in payment. If the total transferred amount does not correspond to the expected amount according to contract rules, then the contract goes to a specific state \(`Defaulted`\).

Underlying's values are transferred and logged to the contract in order to compute expected payment. These values are signed by the oracle \(typically the exchange institution\).

The contract has 2 actions :

* `pay_note` : the issuer uses this action to transfer currency to the contract
* `add_fixing` : logs underlying values

and 4 transitions :

* confirm : the owner has transferred the nominal to the contract
* cancel : the owner can cancel the contract
* check : the owner can check transferred amount
* terminate : finalises contract state when everything is ok

Below is the contract state machine diagram :

![auto-callable state diagram](../../.gitbook/assets/autocallable_diag.png)



{% tabs %}
{% tab title="autocallable.arl" %}
```ocaml
archetype autocallable

constant issuer role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk
constant owner  role = @tz1uNrUbontHGor25CztkKksC8RvjUWAbXYJ
constant oracle role = @tz1r25CztkKksC8RvjuNrUWAbXYJUbontHGo (* exchange *)

constant nominal tez = 1000tz

constant trade      date = 2017-03-14T00:00:00
constant init       date = 2017-03-14T00:00:00
constant issue      date = 2017-03-28T00:00:00
constant final      date = 2020-03-16T00:00:00
constant redemption date = 2020-03-30T00:00:00

(* UNDERLYINGS *)
constant bac_initial rational = 25.32
constant sg_initial  rational = 46.945
constant ubs_initial rational = 15.98

constant bac_strike rational = 12.66   (* ~ 0.5 * bac_initial *)
constant sg_strike  rational = 23.4725 (* ~ 0.5 * sg_initial  *) 
constant ubs_strike rational = 15.98   (* ~ 0.5 * ubs_initial *)

(* CONTRACT DATA *)
asset early identified by observation = {
  observation : date;
  redemption  : date;
  trigger     : rational;
  value       : rational;
} initialized by [
  { 2018-03-14T00:00:00; 2018-03-28T00:00:00; 0.95; 1 };
  { 2018-06-14T00:00:00; 2018-06-28T00:00:00; 0.95; 1 };
  { 2018-09-14T00:00:00; 2018-09-28T00:00:00; 0.95; 1 };
  { 2018-12-14T00:00:00; 2019-01-02T00:00:00; 0.95; 1 };
  { 2019-03-14T00:00:00; 2019-03-28T00:00:00; 0.80; 1 };
  { 2019-06-14T00:00:00; 2019-06-28T00:00:00; 0.80; 1 };
  { 2019-09-16T00:00:00; 2020-09-30T00:00:00; 0.70; 1 };
  { 2019-12-16T00:00:00; 2020-01-02T00:00:00; 0.70; 1 };
  { 2020-03-16T00:00:00; 2020-03-30T00:00:00; 0.70; 1 }
]

asset interest identified by observation = {
  observation : date;
  payment     : date;
  barrier     : rational;
  rate        : rational;
} with {
  i3 : 0 <= barrier <= 1
} initialized by [
  { 2017-06-14T00:00:00; 2017-06-28T00:00:00; 0.5; 2.025  };
  { 2017-09-14T00:00:00; 2017-09-28T00:00:00; 0.5; 4.05   };
  { 2017-12-14T00:00:00; 2018-01-02T00:00:00; 0.5; 6.075  };
  { 2018-03-14T00:00:00; 2018-03-28T00:00:00; 0.5; 8.1    };
  { 2018-06-14T00:00:00; 2018-06-28T00:00:00; 0.5; 10.125 };
  { 2018-09-14T00:00:00; 2018-09-28T00:00:00; 0.5; 12.15  };
  { 2018-12-14T00:00:00; 2019-01-02T00:00:00; 0.5; 14.175 };
  { 2019-03-14T00:00:00; 2019-03-28T00:00:00; 0.5; 16.2   };
  { 2019-06-14T00:00:00; 2019-06-28T00:00:00; 0.5; 18.225 };
  { 2019-09-16T00:00:00; 2019-09-30T00:00:00; 0.5; 20.25  };
  { 2019-12-16T00:00:00; 2020-01-02T00:00:00; 0.5; 22.275 };
  { 2020-03-16T00:00:00; 2020-03-30T00:00:00; 0.5; 24.3   }
]

(* underlyings values *)
asset fixing identified by observation = {
  observation : date;
  bac : rational;  (* Bank of America Corporation *)
  sg  : rational;  (* Société Générale *)
  ubs : rational;  (* Union des Banques Suisses *)
}

(* EXPECTED PAYMENT COMPUTATION *)

function compute_expected (d : date) : tez = {

  verification {

    (** etrigger is defined as the set of early assets for which 
        the trigger condition is true *)
    definition etrigger = { e : early |
      forall f : fixing, 
        if e.obs = f.obs
        then (* trigger condition *)
              f.bac >= e.trigger * bac_initial 
          and f.sg  >= e.trigger * sg_initial
          and f.ubs >= e.trigger * ubs_initial
    }

    (** ibarrier is defined as the set of interest assets for which 
        the barrier condition is true *)
    definition ibarrier = { i : interest | 
      forall f : fixing,
        (* retrieving the first element of etrigger *)
        let efirst = first etrigger in 
          if i.obs = f.obs and i.obs <= efirst.obs
          then (* barrier condition *)
                    f.bac >= bac_strike 
                and f.sg  >= sg_strike
                and f.ubs >= usb_strike
        otherwise interest
    }

    specification {
      (** expected is the sum of redemption nominal and interests *)
      p_expected : 
        let expected =  
          let ftrigger = first etrigger in
            (* early redemption *)
            ftrigger.value * nominal
          otherwise (* etrigger is empty, no early redemption *)
          (* redemption *)
            let f = fixing.get redemption in
            if    f.bac >= bac_strike 
              and f.sg  >= sg_strike
              and f.ubs >= usb_strike
            then
              nominal
            else
              let bac_trigger = f.bac / bac_strike in
              let sg_trigger  = f.sg  / sg_strike  in
              let ubs_trigger = f.ubs / ubs_strike in 
              let worst = min (min bac_trigger sg_trigger) ubs_trigger in
              worst * nominal
        in
        (* interests *)
        let interests =
          let lbarrier = last ibarrier in
            let v = fixing.get i.observation in
            if    v.bac >= i.barrier * bac_initial 
              and v.sg  >= i.barrier * sg_initial
              and v.ubs >= i.barrier * ubs_initial
            then i.rate * nominal
          otherwise 0 
        in
         result = expected + interests 
    }
  }
  
  effect {
    let expected = 0tz in
    let terminated = false in
    let redeem_date = final in
    (* early redemption *)
    redeemloop : for (e in early) (
      if e.redemption <= cd
      then (* is there early redemption ? *)
        let v = fixing.get e.observation in
        if     v.bac >= e.trigger * bac_initial
           and v.sg  >= e.trigger * sg_initial
           and v.ubs >= e.trigger * ubs_initial
        then (
           expected += e.value * nominal;
           redem_date := e.observation;
           terminated := true
        )
      else (* no need to check future observation *)
        break
    );
    (* redemption *)
    if not terminated and redemption <= cd
    then
      let f = fixing.get redemption in
      if     f.bac >= bac_strike
         and f.sg  >= sg_strike
         and f.ubs >= usb_strike
      then
         expected += nominal
      else
         let bac_trigger = f.bac / bac_strike in
         let sg_trigger  = f.sg  / sg_strike  in
         let ubs_trigger = f.ubs / ubs_strike in
         let worst = min (min bac_trigger sg_trigger) ubs_trigger in
         expected += worst * nominal;
    (* expected interests *)
    let exp_interests = 0.0 in
    interestloop : for (i in interest) (
      if i.observation <= redem_date and i.payment <= cd
      then
        let v = fixing.get i.observation in
        if     v.bac >= i.barrier * bac_initial
           and v.sg  >= i.barrier * sg_initial
           and v.ubs >= i.barrier * ubs_initial
        then exp_interests := i.rate * nominal
    );
    expected += exp_interests;
    return expected
  }
}

(* PAYMENT action *)
variable actual_payment tez = 0

action pay_note = {
  called by issuer
  effect {
    actual_payment += transferred
  }
}

action add_fixing (f[%signedby oracle%] : fixing) = {
  effect {
    fixing.add f
  }
}

(* STATE MACHINE *)
states =
 | Created initial (** doc initial state. *)
 | Canceled        (** owner or issuer has canceled the transaction. *)
 | Confirmed       (** owner has confirmed. *)
 | Defaulted
 | Terminated      

(** Used by owner to confirm transaction. It transfers the price of contract (nominal) *)
transition confirm from Created = {
   called by owner
   to Confirmed when { transferred = nominal }
}

transition cancel from Created = {
   called by owner or issuer
   to Canceled
}

transition check from Confirmed = {
  called by owner
  to Defaulted when { actual_payment < compute_expected now }
}

transition terminate from Confirmed = {
  called by issuer
  to Terminated when { actual_payment >= compute_expected now }
}
```
{% endtab %}
{% endtabs %}



The simulation sheet of this contract is available [here](https://docs.google.com/spreadsheets/d/1f_DWUiXUJn9kglnnstkaYP7hMzKt3lyF7YX2L5AQGRY/edit?usp=sharing).

This contract supposes that fixing values are public, which is an issue because banks usually have to pay Exchange institutions a licence to use them. One could easily imagine a contract crawler which would retrieve and consolidate all fixing data.

There are two solutions :

* find a way to cipher fixing data
* provide exchange institutions with a business model which does not rely on fixing data licensing

One could imagine for example that exchange institutions provide fixing data for free and provide SCAAS \(smart contract as a service\); typically this auto-callable note as a service to deploy on the blockchain. 

Note that if underlying values were quoted on the blockchain itself, then the data could be natively public and available. The contract could then rely on calls to the exchange contract.

TODO :

* Search for quotation contracts. Are existing market places a model of that?
* Add formal properties :
  * Sum of payments are above a constant \(here 0, that is proving that payments are positive\)
  * Temporal consistency : adding a fixing at time t does not change the amount due at time before t

