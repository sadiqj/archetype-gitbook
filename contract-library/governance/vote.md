---
description: Basic voting process
---

# Voting process

This archetype defines a basic voting process. The chairman is responsible for registering voters. The use of the `mutable` extension gives the chairman the extra right to modify the voting agenda, in Created state only.

The result of the vote is computed with the `bury` action: winners are ballots with the highest number of votes.

{% tabs %}
{% tab title="voting\_process.arl" %}
```ocaml
archetype vote

variable[%transferable%] chairperson role

(* vote start *)
variable[%mutable chairperson (state = Created)%] startDate date

(* vote deadline *)
variable[%mutable chairperson (state = Created)%] deadline date

asset voter identified by address = {
  address : address;
  hasVoted : boolean
} 

asset ballot identified by value = {
  value   : string;
  nbvotes : uint
}

asset winner = {
  value : string
}

(* state machine *)
states =
 | Created initial  with { s1 : is_empty winner }
 | Voting           with { s2 : is_empty winner }
 | Buried

action register (voter : address) = {
  called by chairperson
  require {
    c1 : state = Created
  }
  effect {
     voter.add { voter; false }
  }
}

transition start from Created = {
   to Vote when { now > startDate }
}

action vote (val : string) = {
   called by voter
   require {
      c2 : state = Voting;
      c3 : (voter.get caller).hasVoted = false
   }
   
   effect {
      (voter.get caller).hasVoted := true;
       ballot.update value { nbvotes += 1 } { nbvotes = 0 }
   }
   
}

transition bury from Voting = {
  require {
    c4 : now > deadline
  }
  to Buried
  with effect {
    let nbvotesMax = ballot.max(nbvotes) in
    for (b in ballot)
      if (b.nbvotes = nbvotesMax)
      then winner.add {value = b.val}
  }
}

specification {
  s3 : startdate < deadline;
  s4 : (voter.select(voter.hasVoted = true)).count() = ballot.sum(nbvotes);
  s5 : forall w : winner,
         forall b : ballot,
           b.nbvotes <= ballot.get(w.value).nbvotes
}
```
{% endtab %}
{% endtabs %}

