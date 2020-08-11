---
description: A contract is operated through entry points.
---

# Entries, Functions

## Entry point

The keyword entry is used to declare an entry point followed by its name.

```javascript
entry main() {
  ...
}
```

If the required entry point name conflicts with a language keyword, you may prefix the id with % to bypass these constraints. For example, `transfer` is a the identifier reserved for the transfer instruction.  The following enables us to declare an entry point named transfer.

```javascript
entry %transfer () {
   ...
}
```

### Arguments

An entry may take arguments. For example, the entry below named "complete" takes two arguments `value` and `amount` , respectively of type `string` and `int`:

```coffeescript
entry complete (value : string, amount : int) {

  ...  

}
```

### Sections

An entry is made of sections listed below:

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
      <td style="text-align:left"><code>refuse transfer</code>
      </td>
      <td style="text-align:left">specifies whether the action refuses incoming
        <br />currency transfer (accepted by default).</td>
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
        <br />(see &quot;specification&quot; section)</td>
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
      <td style="text-align:left">yes</td>
    </tr>
  </tbody>
</table>

For example, the `complete` entry example may be enhanced as follows:

```coffeescript
constant owner : role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk

variable threshold : tez = 10tz

action complete (value : string, amount : int) {
  called by owner
  
  require {
    r : transferred > threshold
  }
  
  effect {
    ... 
  }
}
```

`transferred` is the amount of tez transferred to the entry.

In the expression line 9 above, `r` is a _label_ for the `require` expression \(see [Label](action.md#label) section below\).

### Label

In Archetype, some expressions are named. The following syntax is used to name an expression:

```cpp
lbl : ... expression ... /* lbl is a label */
```

The following is the list of expressions that require a label:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Require</td>
      <td style="text-align:left">
        <p><code>require {</code>
        </p>
        <p><code>  r1 otherwise &quot;NotEnoughTransferred&quot; : transferred &gt; threshold</code>
        </p>
        <p><code>}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Fail if</td>
      <td style="text-align:left">
        <p><code>failif {</code>
        </p>
        <p><code>  f1 with &quot;NotEnoughTransferred&quot; : transferred &lt;= threshold</code>
        </p>
        <p><code>}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Asset invariant</td>
      <td style="text-align:left">
        <p><code>asset mile {</code>
        </p>
        <p><code>  quantity : int</code>
        </p>
        <p><code>} with {</code>
        </p>
        <p><code>  quant_strictly_pos : quantity &gt; 0</code>
        </p>
        <p><code>}</code>
        </p>
        <p>(see <a href="contract-specification/">Contract specification</a>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Security predicate</td>
      <td style="text-align:left">
        <p><code>security {</code>
        </p>
        <p><code>  s1 : only_by_role(anyaction, admin)</code>
        </p>
        <p><code>}</code>
        </p>
        <p>(see <a href="contract-specification/">Contract specification</a>)</p>
      </td>
    </tr>
  </tbody>
</table>

### Effect

#### Local variable 

A local variable is declared as exampled below:

```javascript
var p = amount + 10tz;
```

#### Data assignment

A variable \(global or local\) is assigned a new value as exampled below:

```ocaml
p := amount;
```

Note that is it _not possible_ to assign a value to an input value.

#### Conditional

The basic conditional expression is exampled below :

```ocaml
if transferred > threshold then (
  transfer price to owner;
  transfer 1tz to coder
) else
  fail("not enough");
```

#### Failing

The `fail` instruction interrupts the execution. It reverts the contract storage to its initial state.

```javascript
fail("Error");
```

The `dorequire` expression fails if the condition is not met:

```text
dorequire (balance > threshold,"NotEnoughBalance")
```

The `dofailif` expression fails if the condition is met:

```text
dofailif (balance <= threshold, "NotEnoughBalance")
```

It is possible to pass any Michelson-compliant typed value as the error message to the fail instructions above. 

For example, the following passes the balance and the threshold as additional error message information:

```text
dofailif (balance <= threshold, ("NotEnoughBalance",(balance,threshold)))
```

## Function

It is possible to declare functions with the `function` keyword. The main differences between entries and functions are:

* functions return a value with the `return` keyword
* functions _**cannot**_ modify the contract's storage \(they are "pure"\)

Typically functions help factorize computation codes.

```javascript
function get_rate (amount : tez) : rational {
   var a : int  = amount;
   return (a / 3600);
}
```

Functions are called with the standard call syntax:

```javascript
effect {
   var r = get_rate(transferred);
   if r < 3.14125 then fail("invalid transferred amount");
}
```

