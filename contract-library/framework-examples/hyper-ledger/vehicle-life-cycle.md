# Vehicle life cycle

> This network tracks the Lifecycle of Vehicles from manufacture to being scrapped involving private owners, manufacturers and scrap merchants. A regulator is able to provide oversight throughout this whole process.
>
> This business network defines:
>
> **Participants** `AuctionHouse` `Company` `Manufacturer` `PrivateOwner` `Regulator` `ScrapMerchant`
>
> **Assets** `Order` `Vehicle`
>
> **Transactions** `PlaceOrder` `UpdateOrderStatus` `ApplicationForVehicleRegistrationCertificate` `PrivateVehicleTransfer` `ScrapVehicle` `UpdateSuspiciousScrapAllVehiclesByColour` `SetupDemo`
>
> **Events** `PlaceOrderEvent` `UpdateOrderStatusEvent` `ScrapVehicleEvent`
>
> A `PriavteOwner` participant would submit a `PlaceOrder` transaction, through a Manufacturer's application. A `Manufacturer` would submit an `UpdateOrderStatus` transaction which would be the Vehicle being manufactured. They would apply for a registration certificate by submitting an `ApplicationForVehicleRegistrationCertificate` transaction. After the vehicle has been manufactured they would submit a `PrivateVehicleTransfer` transaction. A `Regulator` would be able perform oversight over this whole process and submit an `UpdateSuspicious` transaction to view any suspicious vehicles that may be out of compliance with regulations. A `ScrapMerchant` would be able to submit a `ScrapVehicle` or a `ScrapAllVehiclesByColour` transaction to complete the lifecycle of a vehicle.

{% embed url="https://github.com/hyperledger/composer-sample-networks/tree/master/packages/vehicle-lifecycle-network" %}

{% tabs %}
{% tab title="vehicle\_lifecycle.arl" %}
```ocaml
archetype vehicle_lifecycle

asset owner = {
  id : role;
  fn : string;
  ln : string
}

enum order_state =
  | Placed                      initial
  | Scheduled_for_manufacture
  | Vin_assigned
  | Owner_assigned
  | Delivered

asset vehicledetail = {
  id : string
}

asset order = {
   status       : order_sate;
   manufacturer : manufacturer;
   orderer      : owner;
   details      : vehicledetail
}

enum vehicle_state =
  | Off_the_road               initial
  | Active
  | Scrapped

asset vehicle identified by vin = {
   vin          : string;
   owner        : owner;
   detail       : vehicledetail;
   color        : string
} with states vehicle_state

action placeOrder (neworder : order) = {
  called by neworder.orderer
  effect {
    order.add neworder
  }
}

transition assign_vin (vin : string) (detail : vehicledetail) on ok : order from Placed = {
  called by order.orderer

  to Vin_assigned
  with effect {
     vehicule.add { vin = vin; detail = detail }
  }
}

transition assign_owner on ok : order from any = {
  to Owner_assigned
  with effect {
    (* set vehicule state *)
    (order.get ok).vehicule.state := Active;
    (* set vehicule owner *)
    (order.get ok).vehicule.owner := order.orderer
  }
}

action vehiculeTransfer (buyer : owner) (vehicule : vehicule) = {
  called by [%delegable%] vehicule.owner
  effect {
    vehicule.owner := buyer
  }
}

transition scrapVehicle on vk : vehicule from (Off_the_road or Active) = {
  called by vehicle.owner

  to Scrapped
}

action scrapVehiclebyColor (color : string) = {
  effect {
    for (v in vehicule.select(vehicule.color = color))
      v.state := Scrapped
  }
}

```
{% endtab %}
{% endtabs %}



