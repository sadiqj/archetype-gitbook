---
description: Transfer decision relies on oracle data
---

# With oracle decision

The escrow with penalty contract is not totally satisfactory because the decision to transfer or cancel is left to the oracle, which is not really the role of an oracle. The oracle is more likely to generate information in pojo-like format. The decision procedure is left to the contract.

In this version, the oracle says whether a task has been completed or not, in the form of a pojo with two informations:

1. a date
2. a status \(`OK` or `KO`\)

The decision rule is the following: if the date is before the deadline and if the status is “OK” then the transfer occurs, otherwise it is cancelled.

The oracle provides with a description of the pojo in the [JSON SCHEMA](https://json-schema.org/) format:

```yaml
{
  "$id": "https://oracle.com/task_status.schema.json",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TaskStatus",
  "type": "object",
  "required": [ "date", “status” ],
  "properties": {
    "date": {
      "type": "date",
      "description": "Date of the task status."
    },
    "status": {
      "type": "string",
      "enum": ["OK", "KO"],
      "description": "Task status."
    }
  }
}
```

The code below illustrates the use of the above schema with the object keyword:

{% tabs %}
{% tab title="escrow\_with\_oracle\_decision.arl" %}
```ocaml
archetype escrow_with_object

variable buyer role

variable[%transferable%] debitor role = buyer

variable seller role

variable[%transferable%] creditor role = seller

variable oracle role

variable[%traceable%] price tez from buyer to creditor

variable[%traceable%] [%mutable_signed [buyer, debitor] (instate Created)%]
     penalty tez from seller to debitor = 0.1 * price

(* action deadline *)
variable[%mutable (buyer or seller) (instate Created)%] deadline date

(* type taskStatus = {
     date : date;
     status : string of [ “OK” | “KO” ]
   }
*)
variable[%signedby oracle%] taskStatus object = "https://oracle.io/tskstat.schema.json"

(* state machine *)
states =
 | Created initial
 | Aborted
 | Confirmed
 | Canceled    with { s1 : balance = 0 }
 | Transferred with { s2 : transferred_to(buyer) = price;
                      s3 : balance = 0;
                      s4 : needs oracle
                    }

transition abort from Created = {
  called by buyer or seller

  to Aborted
}

transition[%signedbyall [{buyer}; {seller}]%] confirm from Created = {
  to Confirmed when { balance = price + penalty }
}

transition finalize (task : taskStatus) from any = {
  to Transferred
  when { task.date <= deadline and task.status }
  with effect {
    transfer price;
    transfer back penalty
  }

  to Canceled
  with effect {
    transfer penalty;
    transfer back price
  }
}

specification {
  s5 : state = Transferred or state = Canceled -> balance = 0;
  s6 : state = Transferred or state = Canceled -> needs oracle;
  s7 : state = Transferred -> receives buyer = price
}


```
{% endtab %}
{% endtabs %}

