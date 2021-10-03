# Conversions

It is often necessary to cast \(convert\) a value from one type to another: for example, say you want to mulitply the `transferred` amount by a duration value to get a new duration value. It is then necessary to convert the tez amount and the duration value into integer values in order to multiply then.

The general rule is that conversions to integer are _implicit_, and conversions back to specific types \(tez, duration\) are not.

One conversion worthy of note is the _not_ explicit conversion from tez to nat: it requires the use of the `mutez_to_nat` function. For example the following converts the transferred amount to a `nat` value:

```javascript
dorequire(transferred = 11tz, "invalid transferred amount");
var t : nat = mutez_to_nat(transferred); // t = 11_000_000
```

The following types may be _implicitly_ converted to integer:

* `duration`
* `date` 

The resulting integer value is the number of the smallest value of the source type: namely second for `duration`, and utez for `tez`.

The following example illustrates this mecanism: 

```javascript
effect {
  var d : int = 1h15m;       // d is the number of seconds (4500)
  var res = t * d * 1s;      // res is a duration
}
```

Note that:

* the conversion form `int` to a `duration` \(line 4\) is done by multiplying `t * d` by `1s`
* the conversion from `rational` to `int` is done with operators `ceil` and `floor`
* the conversion from `int` to `nat` is done with `abs`

