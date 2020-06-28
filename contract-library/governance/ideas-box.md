---
description: Ideas box voting process.
---

# Ideas box

![](../../.gitbook/assets/19421048_s.jpg)  

An ideas box enables to vote for ideas in order to select ideas that appeal to the maximum numbers of voters.

In the process presented here, voters are each granted with a voting weight of 5 that they can attribute to the ideas of their choice: a voter can either vote for 5 ideas each with a weight of 1, or vote for 1 idea with a weight of 5. 

When the voting process is over, the box administrator may declare the selected ideas: the top 3 ideas are selected with the constraint that a selected idea must reach at least 6 votes.

```javascript
archetype ideasbox

constant admin : address = @tz1ZAQXACaEqryobpBoLbJUc2DjG5ZzrhARu
constant expiration : date = 2020-07-22

asset idea {
    id      : int;
    desc    : string;
    nbvotes : int = 0;
}

asset voter {
    addr : address;
    remaining : int = 5;
}

asset winner {
    wid : int;
}

entry register (a_voter : address) { effect { 
    voter.add({addr = a_voter}); 
}}

entry add_idea(description : string) {
    called by admin
    effect { idea.add({ id = idea.count(); desc = description}) }
}

entry vote(n : int, weight : int) {
    require {
        r1 : voter.contains(caller);
        r2 : now <= expiration;
    }
    effect {
       if voter[caller].remaining >= weight then (
            voter[caller].remaining -= weight;
            idea[n].nbvotes += weight;
        )
    }
}

entry finalize() {
    called by admin
    require { r3 : now > expiration }
    effect {
        for i in idea.select(the.nbvotes > 5).sort(desc(nbvotes)).head(3) do
            winner.add({wid = i})
        done
    }
}

specification {
    s1 : idea.sum(nbvotes) = voter.count() * 5 - voter.sum(remaining)
}
```

