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
  | Off_the_road
  | Active
  | Scrapped

asset vehicle identified by vin {
   vin          : string;
   aowner       : pkey of owner;
   detail       : pkey of vehicledetail;
   color        : string;
   vstate       : vehicle_state = Off_the_road;
}

entry placeOrder (iid           : string,
                  imanufacturer : pkey of manufacturer,
                  iorderer      : pkey of owner,
                  idetails      : pkey of vehicledetail) {
  called by iorderer
  effect {
    order.add({ iid; imanufacturer; iorderer; idetails})
  }
}

transition assign_vin (avin : string, adetail : pkey of vehicledetail) on (ok : pkey of order) {
  called by order[ok].orderer

  from Placed
  to Vin_assigned
  with effect {
     vehicle.add ({ vin = avin; aowner = order[ok].orderer; detail = adetail; color = vehicledetail[adetail].dcolor })
  }
}

transition assign_owner () on (ok : pkey of order) {
  from any
  to Owner_assigned
  with effect {
    (* set vehicle state *)
    vehicle[vehicledetail[order[ok].details].idv].vstate := Active;
  }
}

entry vehicleTransfer (buyer : pkey of owner, vk : pkey of vehicle) {
  called by vehicle[vk].aowner
  effect {
    vehicle[vk].aowner := buyer
  }
}

entry scrapVehicle (vk : pkey of vehicle) {
  called by vehicle[vk].aowner
  require {
      r1: vehicle[vk].vstate = Off_the_road or vehicle[vk].vstate = Active
  }
  effect {
      vehicle[vk].vstate := Scrapped
  }
}

entry scrapVehiclebyColor (acolor : string) {
  effect {
    for v in vehicle.select(color = acolor) do
      vehicle[v].vstate := Scrapped
    done
  }
}

```



