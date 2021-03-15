# Coase

The Coase smart contract enables the trading \(buy and sell\) of digital collectible card games:

The smart contract version we are considering here is the one that comes as a Ligo example:

{% embed url="https://medium.com/tezoscommons/a-digital-collectible-card-game-coming-soon-to-tezos-ddd939f3b119" %}

The smart contract version we are considering here is the one that comes as a Ligo example \(_Copyright Coase, Inc 2019_\):

{% embed url="https://gitlab.com/ligolang/ligo/blob/dev/src%2Ftest%2Fcontracts%2Fcoase.ligo" %}

The Archetype version:

```coffeescript
archetype coase

asset card_pattern identified by card_pattern_id {
  card_pattern_id: int;
  coefficient: tez;
  quantity: int;
} shadow {
  sellcount: int = 0;
} with {
  cp1: card_pattern_id >= 0;
  cp2: quantity >= 0;
}

asset card identified by card_id {
  card_id: int;
  card_owner: address;
  card_pattern_card: pkey<card_pattern>;
} with {
  c1: card_id >= 0;
}

variable next_id : int = 0
with {
  n1: next_id >= 0;
  n2: forall c in card, next_id <> c.card_id;
}

entry transfer_single (card_to_transfer : pkey<card>, destination : address) {
  require {
    ts1: caller = card[card_to_transfer].card_owner;
  }
  effect {
    card.update(card_to_transfer, {card_owner = destination})
  }
}

entry sell_single (card_to_sell : pkey<card>) {
  specification {
    shadow effect {
      card_pattern.update(card[card_to_sell].card_pattern_card, {sellcount += 1})
    }
  }
  require {
    ss1: caller = card[card_to_sell].card_owner;
  }
  effect {
    var cpc = card[card_to_sell].card_pattern_card;
    card_pattern.update(cpc, {quantity -= 1});
    card.remove(card_to_sell);
    var price : tez = card_pattern[cpc].quantity * card_pattern[cpc].coefficient;
    var receiver = caller;
    transfer price to receiver
  }
}

entry buy_single (card_to_buy : pkey<card_pattern>) {
  specification {
    postcondition bs1 {
      card.count() = before.card.count() + 1
    }
    postcondition bs2 {
      let some cp = card_pattern[card_to_buy] in
      balance = before.balance + cp.quantity * cp.coefficient
      otherwise true
    }
    postcondition bs3 {
      card.select(card_owner = caller).count() >= 1
    }
  }
  accept transfer
  effect {
    var price : tez = (card_pattern[card_to_buy].quantity + 1) * card_pattern[card_to_buy].coefficient;
    dofailif (price > transferred);
    card_pattern.update(card_to_buy, {quantity += 1});
    card.add({card_id = next_id; card_owner = caller; card_pattern_card = card_to_buy});
    next_id += 1
  }
}

specification {
  g1: balance = card_pattern.sum(((quantity * (quantity + 1)) / 2 + sellcount) * coefficient)
}

```

The Archetype contract defines two assets: _card\_pattern_ which holds the number of cards in stock and _card_ which hods its owner and its pattern.

We note that the concept of _asset collection_ makes the code very straightforward compared to lower-level code: for example, the _transfer\_single_ entry point sets the _card\_owner_ field of the card with id _card\_to\_transfer_. 

Below the LIGO version:

```text
const cards : cards = s.cards ;
const card : card = get_force(action.card_to_transfer , cards) ;
if (card.card_owner =/= source) 
then failwith ("This card doesn't belong to you") 
else skip ;
card.card_owner := action.destination ;
cards[action.card_to_transfer] := card ;
s.cards := cards ;
```

Below the Archetype version:

```text
require {
  ts1: caller = card.get(card_to_transfer).card_owner;
}
effect {
  card.update(card_to_transfer, {card_owner = destination})
}
```

We note that, in Archetype, it is possible for example to provide the formula of the balance of the contract:

```text
specification {
  g1: balance = card_pattern.sum(coefficient * ((quantity * (quantity + 1)) / 2 + sellcount))
}
```

which writes mathematically as:

![](https://cdn-images-1.medium.com/max/1600/1*4OcbCg37HGfuGkBLb2vxtQ.png)

where _sellcount_ is the number of times a card has been sold. Note that _sellcount_ is not computed in the original contract. It is defined in Archetype as a s_hadow field_ \(see “Asset shadow field” section below\).

This property makes the contract’s business model explicit: each time a card is sold, the contract earns the value of the card’s coefficient; when every card has been sold \(∀c,quantity\(c\)=0\), the following relation stands:

![](https://cdn-images-1.medium.com/max/1600/1*QX8A4kgmHaIQkBsj2QEkPg.png)

We see that a high-level non-trivial property is a very powerful way to convey information about the contract.



