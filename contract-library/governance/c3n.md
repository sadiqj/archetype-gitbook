# C3N

A French government authority \(cybercrime unit\) deployed a smart contract on Tezos.

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

For those who are not fluent in stack machine, here is a transcription to Archetype:

```javascript
archetype c3n

/* michelson source : https://better-call.dev/main/KT1Gbu1Gm2U47Pmq9VP7ZMy3ZLKecodquAh4/script */

variable admins : list<address> = [caller]

variable hash : bytes = 0x050100000009617263686574797065

entry register (newadmins : option<list<address>> , oldhash : bytes, newhash : bytes) {
    require {
        r1: oldhash = hash;
        r2: contains(admins,caller);
    }
    effect {
        hash := newhash;
        if issome(newadmins) then admins := opt_get(newadmins)
    }
}
```

