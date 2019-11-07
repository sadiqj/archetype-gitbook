---
description: Real life insurance contract
---

# Fizzy

The [fizzy](https://fizzy.axa/fr/) contract is an insurance against flight delay issued by the [Axa](https://www.axa.com/en/) insurance company :

* the insurance [contract](https://fizzy.axa/fr/static/media/conditions-generales.38af84e2.pdf) \(in french\).
* the ethereum [smart contract](https://etherscan.io/address/0xe083515d1541f2a9fd0ca03f189f5d321c73b872#code)

Below is the direct  transcription of the fizzy contract in archetype.

{% tabs %}
{% tab title="fizzy.arl" %}
```ocaml
archetype fizzy

variable creator role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk 

enum status = 
| Created         
| Before           (* flight landed before the limit      *)
| After            (* flight landed after the limit       *)
| Cancelled        (* cancelled by the user               *)
| FlightCancelled  (* flight cancelled by the air company *)
| Redirected       (* flight redirected                   *)
| Diverted         (* flight diverted                     *)

asset insurance = {
  productid : string;
  limit     : date;
  premium   : tez;
  indemnity : tez;
  stat      : status = Created;
} 

asset[@add creator none] flight identified by id = {
  id         : string;
  insurances : insurance partition;
}

action addflightinsurance (fi : string) (* flight id *)
                          (i : insurance) = {
  called by creator

  effect {
    flight.addifnotexist { id = fi };
    let f = flight.get id in
    f.insurances.add i
  }
}

(* data should be signed by oracle ... *)
action updatestatus (fi : string) (arrival : date) = {
  called by creator

  effect {
    let f = flight.get id in
    for (i in f.insurances)
      match i.status with
      | Created -> 
         if arrival > i.limit
         then i.status := After
      | _ -> none
      end
  }
}

action manual (fi : string) (pr : string) (newst : status) = {
   
  called by creator

  effect {
    let f = flight.get fi in
    for (i in f.insurances.select (product = pr))
      match i.status with
       | Created -> i.status := newst
       | _ -> none
      end
  }
}

specification {
   (* any action on storage is performed only by the owner *)
   s1 : anyaction may be performed only by role owner;

   (* this contract does not transfer any tez *)
   s2 : transfers_by_tx anytx = 0;

   (* transaction “updatestatus” is not the only one to potentially 
      perform an update of insurance status *)
   s3 : (update insurance.stat) may be performed only by action
                                                (updatestatus or manual)}
```
{% endtab %}
{% endtabs %}

