---
description: Date and durations help to deal with date related business logic.
---

# Dates, Durations

## Durations

A duration value may be defined with a combination of several duration units:

* second with `s`
* minute with `m`
* hour with `h`
* day with `d`
* week with `w`

```javascript
effect {
  var d1 = 4d;  // 4 days
  var d2 = 1w;  // 1 week
  var d3 = 2d10h // 2 days and 10 hours
} 
```

Durations values may be compared with the 6 comparison operators `= < > <= >= <>`. 

Two duration values may be added or subtracted. It is also possible to multiply a duration by an integer value.

```javascript
effect {
  var d1 = 55w; // 55 weeks
  var d2 = 3d;  // 3 days
  var d3 = d1 + d2 // equal to '55w3d'
  var d2 = 10*d3 // 555 weeks
}
```

## Dates





