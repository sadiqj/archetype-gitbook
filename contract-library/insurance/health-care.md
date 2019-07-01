# Health care

This contract is adapted from the DAML 's  [introductory insurance contract](https://docs.daml.com/getting-started/introduction.html).

DAML is a distributed private ledger. As such it cannot enforce payments and the contract just generates bills of insurer and patient for accounting purposes. 

The strength of a public blockchain with a crypto money, is that it allows payments and accounting of payments in a single business process. The main differences between the contract below and the DAML version are:

* payments are executed through contract entries
* the contract enters the _Running_ state with validation from both insurer and patient parties

The main benefit of this version is that it may serve as an opposable document in the case of a default in payment: indeed, since the contract is used to channel payments, the contract is accounting for any default of payment \(in the _debts_ variables\).

For example, a doctor may go to court and exhibit the _debt_ value of his corresponding doctor asset as a proof of the default in payment.

From a design point of vue, the contract manages several doctors. A doctor is registered with approvals from both patient and insurer parties.

{% hint style="info" %}
We see in this basic example, that a smart contract on a public blockchain with a crypto money, enables organisations to capture the critical aspects of a business process : payments, permissions and accounting.
{% endhint %}

```ocaml
archetype health_care

constant insurer role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk
constant patient role = @tz1uNrUbontHGor25CztkKksC8RvjUWAbXYJ

constant fee tez = 100tz
constant deductible tez = 500tz
constant period duration = 30D

variable last_fee date

variable consultation_debt tez = 0tz

asset doctor identified by id = {
    id   : role;
    debt : tez = 0tz;
}

states =
| Created   initial
| Running
| Canceled

transition[%signedbyall [patient; insurer]%] confirm from Created = {
    (*signed by [insrurer; patient]*)
    to Running
    with effect {
        last_fee := now
    }
}

transition cancel from Created = {
    called by insurer or patient
    to Canceled
}

action[%signedbyall [patient; insurer]%] register_doctor (docid : address) = {
    (*signed by [insurer; patient]*)
    require { r1 : state = Running }
    effect {
        doctor.add { id = docid }
    }
}

action declare_consultation (v : tez) = {
    called by doctor  (* <-> require { doctor.contains caller } *)

    require { r2 : state = Running }
    effect {
        doctor.update caller { debt += v };
        consultation_debt += min v deductible
    }
}

action pay_doctor (docid : address) = {
    verification {
      s1 : balance = before balance
    }
    called by insurer
    accept transfer
    require { r3 : state = Running }
    effect {
        let debt = (doctor.get docid).debt in
        let decrease = min transferred debt in
        transfer decrease to docid;
        transfer (transferred - decrease) to insurer;
        doctor.update docid { debt -= decrease }
    }
}

action pay_fee = {
    verification {
        s2 : balance = before balance
    }
    called by patient
    accept transfer
    require { r4 : state = Running }
    effect {
        let nb_periods = (now - last_fee) / period in  (* div is euclidean *)
        let due = nb_periods * fee in
        let decrease = min transferred due in
        transfer decrease to insrurer;
        transfer (transferred - decrease) to patient;
        last_fee := last_fee + period * nb_periods     
    }
}

action pay_consulation = {
    verification {
        s3 : balance = before balance
    }
    called by patient
    accept transfer
    require { r5 : state = Running }
    effect {
        let decrease = min (transferred, consultation_debt) in
        transfer decrease to insurer;
        transfer (transferred - decrease) to patient;
        consultation_debt -= decrease
    }
}

```

\(See the [signedbyall](../../extensions-1/signed-by-all.md) extension\)

