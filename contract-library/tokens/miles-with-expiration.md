---
description: >-
  This contract archetype manages the expiration date of a mile. An expired mile
  cannot be consumed and may be cleared out by the administrator.
---

# Miles with expiration

A mille owner is identified by an address and possesses some miles. A mile is identified by a string id and has an expiration date. Every mile is possessed by one and only one owner:

![basic model](../../.gitbook/assets/screenshot-2019-10-17-at-10.01.07.png)

A mile can be added and consumed if its expiration date is in the future. 

A storage optimisation is added to the above model. A mile is augmented with a _quantity_ value to gather several miles with the same expiration date. For example, 3 miles with the same expiration date are merged into one mile with the _quantity_ value set to 3:

![3 miles with same expiration date ...](../../.gitbook/assets/screenshot-2019-10-17-at-10.09.02.png)

![... modeled as one mile with a quantity value](../../.gitbook/assets/screenshot-2019-10-17-at-10.10.10.png)

This optimisation comes with a cost of algorithmic complexity in the _consume_ entry \(line 74\).

All entries are called by the _admin_ role, which is ensured by security predicate _g1_ \(line 120\).

{% tabs %}
{% tab title="miles\_with\_expiration.arl" %}
```javascript
archetype miles_with_expiration

variable admin : role = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg

(* id is a string because it is generated off-chain *)
asset mile identified by id {
   id         : string;
   amount     : int;
   expiration : date
} with {
  m1 : amount > 0;
}

(* a partition ensures there is no direct access to mile collection *)
asset owner identified by addr {
  addr  : role;
  miles : partition<mile> = [] (* injective (owner x mile) *)
}

entry add (ow : address, newmile_id : string, newmile_amount : int, newmile_expiration : date) {
   called by admin

   require {
     c1 : newmile_amount > 0;
   }

   failif {
     c2 : mile.contains(newmile_id);
   }

   effect {
     owner.addupdate (ow, { miles += [{id = newmile_id; amount = newmile_amount; expiration = newmile_expiration} ] })
   }
}

entry consume (a : address, quantity : int) {

  specification {

    assert p1 {
      remainder = 0
    }

    postcondition p2 {
      mile.sum(the.amount) = before.mile.sum(the.amount) - quantity
      invariant for loop {
        0 <= remainder <= toiterate.sum(the.amount);
        before.mile.sum(the.amount) = mile.sum(the.amount) + quantity - remainder
      }
    }

    postcondition p3 {
      forall m in removed.mile, m.expiration >= now
      invariant for loop {
        removed.mile.subsetof(by_expiration)
      }
    }

    postcondition p4 {
      added.mile.isempty()
      invariant for loop {
        added.mile.isempty()
      }
    }
  }

  called by admin

  require {
    r2 : quantity >= 0;
  }

  effect {
    var by_expiration = owner[a].miles.sort(expiration).select(the.expiration > now);
    dorequire (by_expiration.sum(the.amount) >= quantity);
    var remainder = quantity;
    for : loop m in by_expiration do
      if remainder > 0
      then (
        if mile[m].amount > remainder
        then (
          mile.update(m, { amount -= remainder });
          remainder := 0
        )
        else if mile[m].amount = remainder
        then (
          remainder := 0;
          owner[a].miles.remove(m)
        ) else (
          remainder -= mile[m].amount;
          owner[a].miles.remove(m)
        )
      )
    done;
    assert p1
  }
}

entry clear_expired () {
  specification {
    postcondition s3 {
      forall m in removed.mile, m.expiration < now
      invariant for loop2 {
        forall m in removed.mile, m.expiration < now
      }
    }
  }

  called by admin

  effect {
    for : loop2 o in owner do
      owner[o].miles.removeif(the.expiration < now)
    done
  }
}

security {
  (*  this ensures that any mile was added with the 'add' entry *)
  g1 : only_by_role (anyentry, admin);
  g2 : only_in_entry (remove (mile), [consume or clear_expired]);
  g3 : not_in_entry (add (mile), consume);
  g4 : no_storage_fail (add)
}

```
{% endtab %}
{% endtabs %}

