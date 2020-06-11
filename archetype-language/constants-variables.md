# Constants, Variables

## Constant

A constant is a global value that cannot be changed. The following snippet defines a global constant named `price` typed `tez` equal to 10 tez:

```ocaml
constant price : tez = 10tz
```

The following defines a `string` typed constant named `msg` :

```ocaml
constant msg : string = "Hello world"
```

## Variable

A variable is a global value that can be changed. The following snippet defines 2 global variables named `seller` and `buyer` typed role equal to a \(here random\) addresses:

```ocaml
variable seller : role = @tz1KksC8RvjUWAbXYJuNrUbontHGor25Cztk
variable buyer  : role = @tz1KUbontHGor25CztkksC8RvjUWAbXYJuNr
```



