# Animal tracking

> This is an Animal Tracking Business Network based on UK DEFRA government regulations \([https://www.gov.uk/animal-movement-england](https://www.gov.uk/animal-movement-england)\). Farmers can move animals between farms/fields and the UK government farming regulator has visibility into the locations of all animals and all animal movements between farms.
>
> This business network defines:
>
> **Participants** `Farmer` `Regulator`
>
> **Assets** `Animal` `Business` `Field`
>
> **Transactions** `AnimalMovementDeparture` `AnimalMovementArrival` `SetupDemo`
>
> Each Farmer owns a Business that is identified by a Single Business Identifier \(SBI\). A Farmer owns a set of Animals. A Business owns a set of Fields. A Field contains a set of Animals owned by the Farmer. Animals can be transferred between Farmers or between Fields.

{% embed url="https://github.com/hyperledger/composer-sample-networks/tree/master/packages/animaltracking-network" %}

{% code title="animal-tracking.arl" %}
```ocaml
archetype animal_tracking

enum animal_e =
| Sheep
| Cattle
| Pig
| Other

enum movement_e =
| In_field initial
| In_transit

asset animal_a identified by ida {
  ida      : string;
  typ      : animal_e;
  location : pkey of field_a;
} with states movement_e

asset business_a identified by id {
  id        : string;
  incomings : animal_a aggregate;
}

asset field_a identified by name {
  name     : string;
  business : pkey of business_a;
}

transition transit (fk : string) on (ak : pkey of animal_a) {
  from In_field
  to In_transit
  with effect {
    business_a[field_a[fk].business].incomings.add(ak)
  }
}

transition arrival (toField : pkey of field_a) on (ak : pkey of animal_a) {
  from In_transit
  to In_field
  with effect {
    animal_a[ak].location := toField;
    business_a[field_a[toField].business].incomings.remove(ak)
  }
}

```
{% endcode %}

