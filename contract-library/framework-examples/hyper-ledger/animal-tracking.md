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

{% code-tabs %}
{% code-tabs-item title="animal-tracking.arl" %}
```ocaml
archetype animal_tracking

enum animal =
 | Sheep
 | Cattle
 | Pig
 | Other

enum movement =
 | In_field initial
 | In_transit

asset animal identified by id = {
 id       : string;
 status   : movement;
 typ      : animal;
 location : field
}

asset field identified by name = {
  name     : string;
  business : business
} with states movement

asset business identified by id = {
  id              : string;
  incomingAnimals : animal set
}

transition transit (fk : string) on ak : animal from In_field = {
  to In_transit
  with effect {
    (field.get fk).business.incomingAnimals.add (animal.get ak)
  }
}

transition arrival (toField : field) on ak : animal from In_transit = {
  to In_field
  with effect {
    (animal.get ak).field := toField;
    toField.business.incomingAnimals.remove ak
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

