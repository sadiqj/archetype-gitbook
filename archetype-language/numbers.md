---
description: 'Two types of numbers are available in Archetype: integers and rationals'
---

# Numbers

## Integers

Integers are defined as follows:

```text
effect {
  var n1 := 345;
  var n2 := -5;
  var n3 := 99999999999999999999999999999999999999999999999999999999999999999;
}
```

 Integer values are _big integers,_ meaning there is no real constraint on the value and it can be negative. __

The 4 arithmetics operations `+ * - /` and the 6 comparison operators `= <> < > <= >=` are available.

Note that there is no _nat_ \(only positive\) type in archetype. You may however specify the positivity property on an integer value.

```text
variable amount : int := 5 with {
  s1 : amount >= 0;
}
```

This property may be transcoded to _whyml_ for verification. Note as a consequence that any range property may be specified. 

## Tezis

_Tez_ is the type to specify an amount in Tezos cryptocurrency. 

```text
effect {
  var t1 := 1tz;   // one tezis
  var t2 := 1mtz:  // one 0.001 tezis
  var t3 := 1utz;  // one mutez
}
```

It is more constrained than integers since you can only add and subtract. All comparison operators are available though.

## Rationals

A rational is the quotient or fraction of two integers. You can define a rational with :

* a quotient of two integers
* a floating point notation
* a percentage notation

The following presents an example for each:

```text
effect {
 var r1 := 6/4;   
 var r2 := 1.5;
 var r3 := 150%;
} 
```

`r1` `r2` and `r3` represent the same rational, $$3/2 $$ . These values are  transcoded to the pair of integers `(3,2)`. A fractions is simplified during transcoding process. 

### Operations

Archetype provides the 4 arithmetic operations `+ - * /` and the minus sign on rationals. They all return a rational. For example:

```text
effect {
  var r1 := (8,6);  // transcoded to (4,3)
  var r2 := 1.8;    // transcoded to (9,5)
  var rpl := r1+r2; // will execute to (43,35)
  var rmi := r1-r2; // will execute to (13,35)
  var rmu := r1*r2; // will execute to (36,15)
  var rdi := r1/r2: // will execute to ()
  var rms := -r1    // will execute to () 
}
```

It is also possible to mix integers and rationals:

```text
effect {
  var r := 5/3;
  var n := 4;
  var rtn := r*n; // will execute to (20/3)
  var rpn := r+n; // will execute to (17/3)
}
```

It is also possible to mix rationals and _tezis_ values, in that order. However the result is a value in _tezis_. 

```text
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

```text
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

### Conversion to tez

There is no explicit cast \(conversion operator\) from a rational to a tez value. You may just multiply by 1tez for example.

```text
effect {
  var a := 2.5;
  transfer (a*1tez) from source to dest;
}
```



