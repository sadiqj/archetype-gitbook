---
description: Basic string operations.
---

# Strings

The `string` type comes with 3 operators:

* `length`: returns the length of the string
* `slice`: returns a subset of a string:
  * the first argument is the offset \(starting from 0\)
  * the second argument is the length of the subset
* `concat`: returns the concatenation two strings

```javascript
effect {
  var s1 := "this is ";
  var s2 := "a string";
  if slice(s2,2,6) <> "string" then fail "weird ...";
  if concat(s1,s2) <> "this is a string" then fail "even weirder ..."
  if length(s1) <> 8 then fail "ok I give up!";
}
```

The `string` type supports the 6 comparison operators `= <= >= < > <>`.

