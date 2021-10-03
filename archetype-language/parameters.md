# Parameters

```text
archetype test_parameter(p : nat, n : string)

variable r : nat = 0

entry exec () {
  r := p + length(n)
}
```

if you compile with this command:

```text
$ archetype test_parameter.arl --init '(1, "astring")'
```

the previous contract is equivalent to:

```text
archetype test_parameter

variable p : nat    = 1
variable n : string = "astring"

variable r : nat = 0

entry exec () {
  r := p + length(n)
}
```

You can also prefix a parameter with `const` keyword which generates a constant instead of a variable.

For example:

```text
archetype test_parameter_const(const c : nat)

variable r : nat = 0

entry exec () {
  r := c
}
```

with

```text
$ archetype test_parameter.arl --init '1'
```

is equivalent to:

```text
archetype test_parameter_const

variable r : nat = 0

entry exec () {
  r := 1
}

```

