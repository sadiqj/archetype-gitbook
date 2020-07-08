# Basic containers

## Lists

Lists are built with `[ ; ]`:

```javascript
effect {
  var l = [ 1; 2; 3 ];
}
```

The `count` operator returns the number of elements in the list.

```javascript
effect {
  var c = count([1; 2; 3]);
  if c = 3 then transfer 1tz to coder;
}
```

The `contains` operator returns true if and only if an element is in the list.

```javascript
effect {
  if contains([1; 2; 3], 2) then transfer 1tz to coder; 
}
```

The `nth` operator retrieves the element at index n starting from 0:

```javascript
effect {
   var v = nth(["1"; "2"; "3"], 2);
   if v = "3" then transfer 1tz to coder;
}
```

It is possible to iterate over a list with the `for do done` loop syntax:

```javascript
effect {
  var l = [3; 4; 5];
  for i in l do
    transfer (i*1tz) to coder;
  done; // coder receives 12tz ...
} 
```

It is possible to create a list by prepending an element \(as the first element\) to an existing list with the `prepend`.

```javascript
effect {
   var l = prepend(["second"], "first"); // l is ["first"; "second"]
   if nth(l, 0) = "first" then transfer 1tz to coder;
}
```

## Sets

## Maps

