---
description: >-
  Three types of numbers are available in Archetype: integers, naturals and
  rationals
---

# Numbers

## Naturals

Naturals are positive integers. They are defined as follows:

```javascript
effect {
  var n1 : nat = 345;
  var n2 : nat = 9999999999999999999999999999999999999999999999999999999;
  var n3 : nat = 10_000_000;
}
```

Note that it is possible to structure large numbers by packs of 3 digits using the underscore character.

3 arithmetics operations `+ * %` and the 6 comparison operators `= <> < > <= >=` are available. Note however that the _difference_ operator returns an _integer_ value \(see integers section below\).

```javascript
effect {
   var a : nat = 5;
   var b : nat = 7;
   var c : int = a - b; /* -2 typed as intger */
   var d : int = b - a; /* 2 typed as integer */
}
```

Note that the `div` operator is the _euclidean_ division and returns a nat value.

The `abs` function is used to convert a integer value to a natural value.

```javascript
effect {
    var a = 5i; /* a is typed 'int'*/
    var b = abs(5); /* b is typed 'nat' */
}
```

## Integers

Integers are defined as follows:

```javascript
effect {
  var n1 : int = 3_458i;
  var n2 : int = -5i;
  var n3 : int = 99999999999999999999999999999999999999999999999999999999i;
  var n4 : int = 10_000_000i;
}
```

Note that the literal syntax of positive integers uses the `i` suffix.

Integer values are _big integers,_ meaning there is no real constraint on the value and it can be negative. __

The 5 arithmetics operations `+ * - div %` and the 6 comparison operators `= <> < > <= >=` are available.

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
  var r1 := 8 / 6;    // transcoded to (4,3)
  var r2 := 1.8;      // transcoded to (9,5)
  var rpl := r1 + r2; // will execute to (6,15)
  var rmi := r1 - r2; // will execute to (-7,15)
  var rmu := r1 * r2; // will execute to (4,5)
  var rdi := r1 / r2: // will execute to (20,27)
  var rms := -r1      // will execute to (-4,3) 
}
```

It is also possible to mix integers and rationals:

```javascript
effect {
  var r := 5 / 3;
  var n := 4;
  var rtn := r * n; // will execute to (20/3)
  var rpn := r + n; // will execute to (17/3)
}
```

It is also possible to mix rationals and _tezies_ values, in that order. However, the result is a value in _tezies_. 

```javascript
effect {
  var r := 80%;
  var a := 56tz;
  var res := r * a;
}
```

In the example above, the `res` value is `44800000utz` . The process is the following:

1. convert a to _utez_ \(smallest unit\)
2. compute the rational \(here 4 \* 56000000 / 5 = 224000000 / 5\)
3. execute the euclidean division \(44800000\)

Note that the term `a * r`  is not accepted as `res` value \(line 4 above\): rationals come first.

### Comparison

Rationals are comparable with `= <> < <= > >=` operators.

```javascript
effect {
   var r1 := 3/12;
   var r2 := 0.25;
   var r3 := 0.5;
   if r1 = r2 and r3 > r2 then transfer 1tz to coder;
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
  transfer (a * 1tz) from source to dest;
}
```



