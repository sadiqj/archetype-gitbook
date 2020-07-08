# Composite types

## Options

An option is convenient when a variable may or may not have a value of a given type.

#### Declare 

`some` and`none` are the 2 operators to build an option value.

```css
variable receiver : option<address> = none // to be set later ...

variable message : option<string> = some("there is a message")
```

#### Test 

`issome` and `isnone` are the 2 operators to test whether an option value is none or is some.

```css
entry testopt (a : option<int>) {
  requires {
     r1 : issome(a);
  }
  failif {
     f1 : isnone(a);
  }
  effect {
    ...
  }
}
```

#### Get 

`getopt` is the operator to extract the value from a `some` option value. It _fails_ if the value is `none`.

```css
effect {
  var a := some("a string");
  if issome(a) then (
    if getopt(a) = "a string" then transfer 1tz to coder;
  );
}
```

## Tuple

## Record











