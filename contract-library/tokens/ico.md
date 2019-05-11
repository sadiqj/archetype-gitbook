---
description: A real life ico process
---

# ICO

This is the archetype version of the ICO process set up by [BCDiploma](https://www.bcdiploma.com/) and described at the following address:

{% embed url="https://github.com/VinceBCD/BCDiploma/blob/master/sources/BCDT/contracts/BCDToken/" %}

```ocaml
archetype[%erc20] bcd_ico

(* contribution thresholds *)
variable[%mutable owner (state = Init)] min_contribution tez = 0.001tz

variable[%mutable owner (state = Init)] max_contribution_silver tez = 10tz

(* bcd token data *)

variable[%mutable owner (state = Init)] max_bcd_to_sell int = 100000000

variable[%mutable owner (state = Init)] exchange_rate_tez_bcd int = 13000

(* round caps *)

variable[%mutable owner (state = Init)] soft_cap tez = 1800tz

variable[%mutable owner (state = Init)] presales_cap tez = 3200tz

variable[%mutable owner (state = Init)] round1_cap tez = 3200tz
(* presales_cap + 1600 *)

(* Number tokens sent, eth raised, … *)

variable nb_bcd_sold int = 0

variable nb_tez_raised int = 0

(* Roles *)

role[%transferable] owner

role[%mutable owner (state = Init)] whitelister 

role[%mutable owner (state = Init)] reserve 
role[%mutable owner (state = Init)] community 
role[%mutable owner (state = Init)] vested 

(* contributor *)

enum whitelist = 
 | Silver
 | Gold

asset[@add @remove @update owner (state = Init)] 
     contributor identified by id = {
   id           : address;
   list         : whitelist;
   contrib      : tez = 0;
} with { 
   i1 : list = Silver -> contrib <= max_contribution_silver;
}

enum[%mutable_enum owner][%incr] phase = 
| Init initial     
| PresaleRunning   with { i2 : nb_tez_raised <= presales_cap }
| PresaleFinished  
| Round1Running    with { i3 : nb_tez_raised <= round1_cap  }
| Round1Finished   
| Round2Running    
| Round2Finished   
| End              (* at the end, no token can be created *)
                   with { i4 : not (exists tx, 
                               (add tokenHolder or update tokenHolder) 
                                            may be performed only by tx tx
                   )}

function is_running = {
  match phase with
  | PresalesRunning | Round1Running | Round2Running -> true
  | _ -> false
  end
}

function get_rate = {
  let coeff = (match phase with
  | PresalesRunning -> 1.2
  | Round1Running   -> 1.1
  | _               -> 1
  end) in
  coeff * exchange_rate_tez_bcd
}

function get_remaining_tez_to_raise = {
  match phase with
  | PresalesRunning | PresalesFinished -> presales_cap - nb_tez_raised
  | Round1Running   | Round1Finished   -> round1_cap - nb_tez_raised
  | _ -> (let remaining_bcd = max_bcd_to_sell - nb_bcd_sold in
          remaining_bcd / exchange_rate_bcd_tez)
  end
}

function transition_to_finished = {
  match phase with
  | PresalesRunning -> PresalesFinished
  | Round1Running   -> Round1Finished
  | Round1Finished  -> Round2Running
  | _               -> Round2Finished
  end
}

action contribute = {

  specification {
    p1 :
      let c = contributor.get caller in
       transferred = transferred_to(caller) + c.contrib - (old c.contrib)
  }

  require {
    c1 : contributor.contains caller;
    c2 : is_running ();
    c3 : transferred >= min_contribution;
    c4 : let c = contributor.get caller in
           not (c.list = Silver and transferred >= max_contrib_silver)
  }
   
  effect {
    (* cap contribution to max_contrib if necessary *)
    let c = contributor.get caller in
    let contrib = transferred in
    if    c.list = Silver
      and c.contrib + contrib >= max_contrib_silver
    then contrib := max_contrib_silver - c.contrib;
    (* cap contribution to round cap if necessary *)
    let remaining_tez = get_remaining_tez_to_raise () in
    if remaining_tez <= contrib
    then ( 
      contrib := remaining_tez;
      phase := transition_to_finished ()
    );
    (* convert contribution to nb of bcd tokens *)
    let rate = get_rate () in
    let nb_tokens  = contrib * rate in
    (* update ico stats *)
    nb_bcd_sold   += nb_tokens;
    nb_tez_raised += contrib;
    (* update caller’s contribution *)
    c.contrib     += contrib; 
    (* syntaxic sugar for
       contributor.update caller { contrib += contrib } *)
    if contrib <= transferred
    then transfer (transferred - contrib) to caller
  }
}

transaction mintAside from Round2Finnished = {
   
   to End
   when {
     is_some reserve and
     is_some community and
     is_some vested;
   }
   with effect … 
}

(* the onlyonce extension specifies that withdraw action can be 
   executed only once, that is a contributor can withdraw only once. *)
action[%onlyonce] withdraw = {

  require { 
     c1 : state = Round2Finished;
     c2 : nb_tez_raised <= soft_cap;
     c3 : (contributor.get caller).contribution > 0
  }
      
  effect {
     let c = contributor.get caller in
     transfer c.contribution to caller;
     (* do not forget this *)
     c.contribution := 0
  }
}

```

