---
description: 'Two types of numbers are available in Archetype: integers and rationals'
---

# Numbers

## Integers

## Rationals

A rational is the quotient or fraction of two integers. You can define a rational with :

* a quotient of two integers
* a floating point notation
* a percentage notation

The following presents an example for each:

```text
effect {
 var r1 := 1.5;   
 var r2 := (6,4);
 var r3 := 150%;
} 
```

`r1` `r2` and `r3` represent the same rational $$3/2 $$ . These values are  transcoded to a pair of integers `(3,2)`.

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

It is possible 



