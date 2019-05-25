# Perishable Goods

> Example business network that shows growers, shippers and importers defining contracts for the price of perishable goods, based on temperature readings received for shipping containers.
>
> The business network defines a contract between growers and importers. The contract stipulates that: On receipt of the shipment the importer pays the grower the unit price x the number of units in the shipment. Shipments that arrive late are free. Shipments that have breached the low temperate threshold have a penalty applied proportional to the magnitude of the breach x a penalty factor. Shipments that have breached the high temperate threshold have a penalty applied proportional to the magnitude of the breach x a penalty factor.
>
> This business network defines:
>
> **Participants** `Grower` `Importer` `Shipper`
>
> **Assets** `Contract` `Shipment`
>
> **Transactions** `TemperatureReading` `ShipmentReceived` `SetupDemo`

{% embed url="https://github.com/hyperledger/composer-sample-networks/tree/master/packages/perishable-network" %}

{% code-tabs %}
{% code-tabs-item title="perishable.arl" %}
```ocaml
archetype perishable

enum productType =
 | Bananas
 | Apples
 | Pears
 | Peaches
 | Coffee

enum shipmentStatus =
 | Created
 | In_transit
 | Arrived

asset grower identified by id = {
  id : role
}

asset importer identified by id = {
  id : role
}

asset importer identified by id = {
  id : role
}

asset p_contract = {
  grower          : grower;
  shipper         : shipper;
  importer        : importer;
  arrivalDateTime : date;
  unitPrice       : rational
}

asset shipment = {
  type     : productType;
  count    : uint;
  p_c      : p_contract
} with states shipmentStatus

transition payOut (arrival : date) on sk : shipment from In_transit = {
  to Arrived
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

