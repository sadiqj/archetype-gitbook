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

{% code title="vehicle\_lifecycle.arl" %}
```ocaml
archetype vehicle_lifecycle

asset owner {
  ido : role;
  fn  : string;
  ln  : string;
}

enum order_state =
  | Placed                      initial
  | Scheduled_for_manufacture
  | Vin_assigned
  | Owner_assigned
  | Delivered

asset vehicledetail {
  idv : string;
  dcolor : string;
}

asset manufacturer {
  mid : string;
}

asset order {
  oid           : string;
  amanufacturer : pkey of manufacturer;
  orderer       : pkey of owner;
  details       : pkey of vehicledetail;
} with states order_state

enum vehicle_state =
  | Off_the_road               initial
  | Active
  | Scrapped

asset vehicle identified by vin {
   vin          : string;
   aowner       : pkey of owner;
   detail       : pkey of vehicledetail;
   color        : string;
} with states vehicle_state

action placeOrder (neworder : order) {
  called by neworder.orderer
  effect {
    order.add(neworder)
  }
}

transition assign_vin (avin : string, adetail : pkey of vehicledetail) on (ok : pkey of order) from Placed {
  called by order.get(ok).orderer

  to Vin_assigned
  with effect {
     vehicle.add ({ vin = avin; aowner = order.get(ok).orderer; detail = adetail; color = vehicledetail.get(adetail).dcolor })
  }
}

transition assign_owner () on (ok : pkey of order) from any {
  to Owner_assigned
  with effect {
    (* set vehicle state *)
    vehicle.get(order.get(ok)).set_state(Active)
  }
}

action vehicleTransfer (buyer : pkey of owner, vk : pkey of vehicle) {
  called by [%delegable%] vehicle.get(vk).aowner
  effect {
    vehicle.get(vk).aowner := buyer
  }
}

transition scrapVehicle () on (vk : pkey of vehicle) from (Off_the_road or Active) {
  called by vehicle.get(vk).aowner

  to Scrapped
}

action scrapVehiclebyColor (acolor : string) {
  effect {
    for v in vehicle.select(color = acolor) do
      v.set_state(Scrapped)
    done
  }
}

```
{% endcode %}



