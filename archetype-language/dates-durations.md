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

Duration values are integers, positive, or negative.

```javascript
effect {
  var d1 = 4d;    // 4 days
  var d2 = 1w;    // 1 week
  var d3 = 2d10h; // 2 days and 10 hours
  var d4 = -5d;   // minus 5 days
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

Date format is [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). 

```javascript
constant date0 : date = 2019-01-01                
constant date1 : date = 2019-01-01T01:02:03       
constant date2 : date = 2019-01-01T01:02:03Z      
constant date3 : date = 2019-01-01T00:00:00+01:00 
constant date4 : date = 2019-01-01T00:00:00-05:30 
```

The subtraction of two dates returns a duration; since the duration is a positive number, if `d1` and `d2` are two dates, then `d1 - d2 = d2 - d2 >= 0` holds.

```javascript
effect {
  var d1 := 2020-06-28;
  var d2 := 2020-05-28;
  if (d1 - d2 <= 4w) then fail("really? ...");
}
```

It is possible to add or subtract a duration to a date value to get a new date.

```javascript
effect {
  var d1 := 2020-06-17;
  if (d1 + 5d < 2020-06-22) then fail("oops I did it again");
}
```

### Now

`now` is the keyword to refer to date at execution.

```javascript
variable deadline : date = 2020-09-01

action submit() {
  require {
    r1 : deadline - 8w <= now <= dealine + 1d
  }
  effect {
    ...
  }
 }
```



