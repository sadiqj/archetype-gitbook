---
description: 'Read, compute and write data.'
---

# Action

Contract data is read and written with actions. They are the entry points of the contract.

## Arguments

An action may take input arguments. For example the snippet below declare an action named `complete` which takes two arguments `value` and `amount` , respectively of type `string` and `int`:

```ocaml
action complete (value : string) (amount : int) = {

  ...  (* action body *)

}
```

## Sections

An Action is made of sections listed below:

<table>
  <thead>
    <tr>
      <th style="text-align:left">name</th>
      <th style="text-align:left">description</th>
      <th style="text-align:left">optional</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>called by</code>
      </td>
      <td style="text-align:left">specifies which role(s) may call the action</td>
      <td style="text-align:left">yes</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>accept transfer</code>
      </td>
      <td style="text-align:left">specifies whether the action accepts incoming
        <br />currency transfer (refused by default).</td>
      <td style="text-align:left">yes</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>require</code>
      </td>
      <td style="text-align:left">lists the required condition to pass in order to
        <br />execute the action.</td>
      <td style="text-align:left">yes</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>specification</code>
      </td>
      <td style="text-align:left">specifies formal properties the action effect has
        <br />(see next section)</td>
      <td style="text-align:left">yes</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>effect</code>
      </td>
      <td style="text-align:left">
        <p>codes the effect of the action:</p>
        <ul>
          <li>changes in data storage</li>
          <li>transfers of currencies</li>
        </ul>
      </td>
      <td style="text-align:left">no</td>
    </tr>
  </tbody>
</table>For example, the `complete` action example may be enhanced as follows:

```ocaml
constant owner role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk

variable threshold tez = 100tz

action complete (value : string) (amount : int) {
  called by owner
  
  accept transfer
  
  require {
    r : transferred > threshold
  }
  
  effect {
    ... (* effect code *)
  }
}
```

In the expression `r : ...` line 11 above, `r` is a label for the _require_ expression \(see Label section below\).

## Effect

### Local variable 

A local variable is declared as exampled below:

```ocaml
let p = amount + 10tz in
...
```

### Data assignment

A variable \(global or local\) is assigned a new value as exampled below:

```ocaml
p := amount;
```

After this instruction, the value of `threshold` is the value of `amount`.

### Currency transfer

The instruction to transfer currency is exampled below:

```ocaml
transfer 10tz to owner
```

The recipient of the transfer may be omitted when it is specified in the tez value declaration. Consider the following declaration:

```ocaml
constant price from buyer to seller = 50tz
```

The transfer of price to seller is simply executed with:

```text
transfer price;
```

The transfer to buyer is simply executed with:

```text
transfer back price;
```

### Conditional

The basic conditional expression is exampled below :

```ocaml
if transferred > threshold then (
  transfer price
) else (
  fail "not enough"
)
```

The `require` expression fails if the condition is not met:

```text
require (transferred > threshold);
```

The `failif` expression fails if the condition is met:

```text
failif (transferred <= threshold)
```

### Asset collection

Consider the following car asset identified by its vin id:

```text
asset car identified by vin = {
  vin : string;
  model : string;
  year : int
}
```

The following table gives the basic instructions to get, add, remove, update an asset. 

| operation | expression |
| :--- | :--- |
| get an asset | `car.get vid` |
| add an asset | `car.add { "1GNEK13ZX3R298984"; "Bugatti Chiron"; "2018" }`  |
| remove an asset | `car.remove vid` |
| update an asset | `car.update "1GNEK13ZX3R298984" { year = 2019 }` |
| retrieve asset nb. i | `car.nth i` \(an asset collection is sorted\) |

Advanced operations over a collection are listed in the table below:

| operation | expression |
| :--- | :--- |
| count | `car.count`  |
| maximum of a field | `car.max { year }` |
| minimum of a field | `car.min { year }` |
| sum of a field | `car.sum { year }` |
| select a subset collection | `car.select { year >= 2019 }`  |
| sort a collection | `car.sort { year }` |

_Iteration_ over a collection is as follows:

```ocaml
for (c in car) (
  ...
)
```



