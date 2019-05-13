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
constant admin role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk

constant threshold tez = 100tz

action complete (value : string) (amount : int) = {
  called by admin
  
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

### Currency transfer

### Condition

#### if then else

#### failif

#### Require

### Asset collection

#### Aggregation

#### Filter

#### Loop





