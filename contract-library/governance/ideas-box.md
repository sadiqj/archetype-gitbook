---
description: Ideas box voting process.
---

# Ideas box

![](../../.gitbook/assets/19421048_s.jpg)  

An ideas box enables to vote for ideas in order to select the ideas that appeal to the maximum numbers of voters.

In the process presented here, voters are each granted with 5 voting tokens that they can attribute to the ideas of their choice: a voter can either vote for 5 ideas each with 1 token, or vote for 1 idea with 5 tokens. 

When the voting process is over, the box administrator may declare the selected ideas: the top 3 ideas are selected with the constraint that a selected idea must reach at least 6 votes.

```javascript
archetype ideasbox

constant chairman : address = @tz1ZAQXACaEqryobpBoLbJUc2DjG5ZzrhARu
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

asset selected {
    wid : int;
}

entry register (a_voter : address) {
    called by chairman
    effect { voter.add({addr = a_voter}) }
}

entry add_idea(description : string) {
    require {
        r0 : now <= expiration - 2d;
    }
    effect {
        idea.add({ id = idea.count(); desc = description})
    }
}

entry vote(n : int, weight : int) {
    specification {
        p1 : let some v = voter[caller] in
             let some bv = before.voter[caller] in
                v.remaining = bv.remaining - weight
             otherwise true otherwise true
    }
    require {
        r1 : voter.contains(caller);
        r2 : expiration - 2d < now <= expiration;
        r3 : voter[caller].remaining >= weight;
    }
    effect {
        voter[caller].remaining -= weight;
        idea[n].nbvotes += weight;
    }
}

entry finalize() {
    called by chairman
    require {
        r4 : now > expiration;
        r5 : selected.count() = 0; /* cannot finalize twice */
    }
    effect {
        for i in idea.select(the.nbvotes >= 5).sort(desc(nbvotes)).head(3) do
            selected.add({i})
        done
    }
}

specification {
    i1 : 5 * voter.count() = idea.sum(nbvotes) + voter.sum(remaining)
}


```

