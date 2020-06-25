# C3N

Last month saw the first smart contract deployed on Tezos by a government authority \(French cybercrime unit\)

{% embed url="https://beincrypto.com/tezos-smart-contracts-used-by-french-army-since-september/" %}

Below is the Michelson of the contract \(see it on [better call dev](https://better-call.dev/main/KT1Gbu1Gm2U47Pmq9VP7ZMy3ZLKecodquAh4/script)\): 

```text
{
{ { DUP ; CAR ; DIP { CDR } } ; { DUP ; CAR @newadmin ; DIP { CDR } } ; DIP { { DUP ; CAR @oldhash ; DIP { CDR @newhash } } } ; DIP { DIP { DIP { { DUP ; CAR @storedadmin ; DIP { CDR @storedhash } } } } } } ;
SWAP ;
{ DIP { DIP { DIP { SWAP } } } } ;
{ DIP { DIP { SWAP } } } ;
DIP { SWAP } ;
{ DIP { DIP { DIP { SWAP } } } } ;
{ DIP { DIP { SWAP ; DUP ; DIP { SWAP } } } } ;
{
{ COMPARE ; EQ } ;
IF { }
$ELSE { { UNIT ; FAILWITH } }
} ;
SENDER ;
SWAP ;
{ DIP { DIP { PUSH @admin bool false } } } ;
ITER { DIP { DUP } ; { COMPARE ; EQ } ; SWAP ; DIP { OR @admin } } ;
DROP ;
{
IF { }
$ELSE { { UNIT ; FAILWITH } }
} ;
IF_NONE { }
$ELSE { DIP { DROP } } ;
NIL operation ;
{ DIP { PAIR %admin %hash } ; PAIR %op }
}
```

For those who are not fluent in stack machine, here is the transcription to Archetype:

```text
archetype c3n

/* michelson source : https://better-call.dev/main/KT1Gbu1Gm2U47Pmq9VP7ZMy3ZLKecodquAh4/script */

asset admins {
    addr: address;
}

variable hash : string = "initialvalue"

entry register (newadmins : ((pkey of admins) list) option, oldhash : string, newhash : string) {
    specification {
        postcondition p1 {
            before.hash = oldhash
        }
        postcondition p2 {
            hash = newhash
        }
        postcondition p3 {
            before.admins.contains(caller)
        }
    }
    require {
        r1: oldhash = hash;
    }
    effect {
        var test = false;
        for admin in admins do
            if admins[admin].addr = caller then
            test := true
        done;
        require (test = true);
        hash := newhash;
        if issome(newadmins) then (
            admins.clear();
            for a in getopt(newadmins) do
                admins.add ({ a })
            done
        )
    }
}
```

Quite straightforward, isn’t it?

