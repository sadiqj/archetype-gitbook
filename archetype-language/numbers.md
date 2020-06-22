---
description: 'Two types of numbers are available in Archetype: integers and rationals'
---

# Numbers, Casts

## Integers

Integers are defined as follows:

```javascript
effect {
  var n1 := 345;
  var n2 := -5;
  var n3 := 99999999999999999999999999999999999999999999999999999999999999999;
}
```

 Integer values are _big integers,_ meaning there is no real constraint on the value and it can be negative. __

The 4 arithmetics operations `+ * - div` and the 6 comparison operators `= <> < > <= >=` are available.

Note that de `div` operator is the _euclidean_ division and returns an integer value.

There is no _nat_ \(only positive\) type in archetype. You may however specify the positivity property on an integer value.

```javascript
variable amount : int := 5 with {
  s1 : amount >= 0;
}
```

This property will be transcoded to _whyml_ for verification. Note as a consequence that any range property may be specified. 

## Tezis

_Tez_ is the type to specify an amount in Tezos cryptocurrency. 

```javascript
effect {
  var t1 := 1tz;   // one tezis
  var t2 := 1mtz:  // one 0.001 tezis
  var t3 := 1utz;  // one mutez
}
```

It is more constrained than integers since you can only add and subtract _tez_ values. All comparison operators are available.

## Rationals

A rational is the quotient or fraction of two integers. You can define a rational with :

* a quotient of two integers
* a floating point notation
* a percentage notation

The following presents an example for each:

```javascript
effect {
 var r1 := 6/4;   
 var r2 := 1.5;
 var r3 := 150%;
} 
```

`r1` `r2` and `r3` represent the same rational, $$3/2 $$ . These values are  transcoded to the pair of integers `(3,2)`. A fractions is simplified during transcoding process. 

### Operations

Archetype provides the 4 arithmetic operations `+ - * /` and the minus sign on rationals. They all return a rational. For example:

```javascript
effect {
  var r1 := 8/6;  // transcoded to (4,3)
  var r2 := 1.8;    // transcoded to (9,5)
  var rpl := r1+r2; // will execute to (6,15)
  var rmi := r1-r2; // will execute to (-7,15)
  var rmu := r1*r2; // will execute to (4,5)
  var rdi := r1/r2: // will execute to (20,27)
  var rms := -r1    // will execute to (-4,3) 
}
```

It is also possible to mix integers and rationals:

```javascript
effect {
  var r := 5/3;
  var n := 4;
  var rtn := r*n; // will execute to (20/3)
  var rpn := r+n; // will execute to (17/3)
}
```

It is also possible to mix rationals and _tezis_ values, in that order. However the result is a value in _tezis_. 

```javascript
effect {
  var r := 80%;
  var a := 56tez;
  var res := r*a;
}
```

In the example above, the `res` value is `44800000utz` . The process is the following:

1. convert a to _utez_ \(smallest unit\)
2. compute the rational \(here 3\*56000000/4 = 168000000/4\)
3. execute the euclidean division \(44800000\)

Note that the term `a*r`  is not accepted as `res` value \(line 4 above\): rationals come first.

### Comparison

Rationals are comparable with `= <> < <= > >=` operators.

```javascript
effect {
   var r1 := 3 / 12;
   var r2 := 0.25;
   var r3 := 0.5;
   if r1 <> r2 or r3 < r2 then fail "Huston, we've got a problem";
}
```

It is possible to compare rationals and integers. It is not possible to compare rationals and _tez_ values.

### Conversion to integers

Rational are converted to integers with the `floor` and `ceil` operators with the expected behaviour.

### Conversion to other types

There is no explicit cast \(conversion operator\) from a rational to a tez value, a duration or a date. You may just multiply by 1tez for example.

```javascript
effect {
  var a := 2.5;
  transfer (a*1tez) from source to dest;
}
```

## Casts

It is often necessary to cast \(convert\) a value from one type to another: for example, say you want to mulitply the `transferred` amount by a duration value to get a new duration value. It is then necessary to convert the tez amount and the duration value into integer values in order to multiply then.

The general rule is that conversions to integer are implicit, and conversions back to specific types \(tez, duration\) are not. For example the following converts the transferred amount to an integer value:

```javascript
var t : int = transferred;
```

Note that it is necessary to _explicitely_ type the `t` variable in order to trigger the _implicit_ conversion of `transferred` to integer. The following types may be implicitly converted to integer:

* `duration`
* `date`
* `tez` 

The resulting integer value is the number of the smallest value of the source type: namely second for `duration`, and utez for `tez`.

The following example illustrates this mecanism: 

```javascript
effect {
  var t : int = transferred; // t is the number of utez 
  var d : int = 1h15m;       // d is the number of seconds (4500)
  var res = t * d * 1s;      // res is a duration
}
```

Note that the conversion back to a duration line 4 is done by explicitly multiplying `t*d` by `1s`. 

Note that conversion from rational to integer is done explicitly with operators `ceil` and `floor`.



