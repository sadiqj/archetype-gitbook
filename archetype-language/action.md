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

action complete (value : string) (amount : int) = {
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

## Effect

### Data assignment

A variable is assigned a new value as exampled below:

```ocaml
threshold := amount;
```

After this instruction, the value of `threshold` is the value of `amount`.

### Currency transfer

Transfers of currency use the transfer instruction as exampled below:

```ocaml
transfer 10tz to owner
```

The recipient of the transfer may be omitted when it is specified in the tez value declaration. Given the following declaration:

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

The failif expression fails if the condition is met:

```text
failif (transferred <= threshold)
```

### Asset collection

#### Aggregation

#### Filter

#### Loop





