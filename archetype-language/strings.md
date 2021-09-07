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
* `to_string` : converts a nat value to a string

```javascript
effect {
  var s1 = "this is ";
  var s2 = "a string";
  if slice(s2,2,6) = "string" then transfer 1tz to dev;
  if concat(s1,s2) = "this is a string" then transfer 1tz to dev;
  if length(s1) = 8 then transfer 1tz to dev;
  if to_string(5) = "5" then transfer 1tz to dev;
}
```

The `string` type supports the 6 comparison operators `= <= >= < > <>`.

