---
description: Real life insurance contract
---

# Fizzy

The [fizzy](https://fizzy.axa/fr/) contract was an insurance against flight delay issued in 2018 by the [Axa](https://www.axa.com/en/) insurance company :

* the insurance [contract](https://fizzy.axa/fr/static/media/conditions-generales.38af84e2.pdf) \(in french\).
* the ethereum [smart contract](https://etherscan.io/address/0xe083515d1541f2a9fd0ca03f189f5d321c73b872#code)

Below is the direct  transcription of the fizzy contract in archetype.

{% code title="fizzy.arl" %}
```ocaml
archetype fizzy

variable creator : role = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

enum status =
| Created
| Before           (* flight landed before the limit      *)
| After            (* flight landed after the limit       *)
| Cancelled        (* cancelled by the user               *)
| FlightCancelled  (* flight cancelled by the air company *)
| Redirected       (* flight redirected                   *)
| Diverted         (* flight diverted                     *)

asset insurance {
  productid : string;
  limit     : date;
  premium   : tez;
  indemnity : tez;
  product   : string;
  stat      : status = Created;
}

asset flight identified by id {
  id         : string;
  insurances : insurance partition;
}

entry addflightinsurance (fi : string, iproductid : string, ilimit : date, ipremium : tez, iindemnity : tez, iproduct : string, istat : status) {
    called by creator

    effect {
      if (not flight.contains (fi)) then
      flight.add({ id = fi; insurances = [] });
      flight[fi].insurances.add({iproductid; ilimit; ipremium; iindemnity; iproduct; istat})
    }
}

(* data should be signed by oracle ... *)
entry updatestatus (fi : string, arrival : date) {

    called by creator

    effect {
      for i in flight[fi].insurances do
        match insurance[i].stat with
        | Created ->
           if arrival > insurance[i].limit
           then insurance[i].stat := After
        | _ -> ()
        end
      done
    }
}

entry manual (fi : string, pr : string, newst : status) {

    called by creator

    effect {
      for i in flight[fi].insurances.select(product = pr) do
        match insurance[i].stat with
         | Created -> insurance[i].stat := newst
         | _ -> ()
        end
      done
    }
}

security {
  (* any action on storage is performed only by the owner *)
  s1 : only_by_role (anyentry, creator);
}

```
{% endcode %}

