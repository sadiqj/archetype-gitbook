# Basic containers

## Lists

Lists are built with `[ ; ]`:

```javascript
effect {
  var l = [ 1; 2; 3 ];
}
```

The `length` operator returns the number of elements in the list.

```javascript
effect {
  var c = length([1; 2; 3]);
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

Sets are containers of unique objects. It is possible to add and remove an object, test if an object is in a set, and know the number of elements in a set. 

Sets may currently contain only builtin types: `int` `nat` `string` `bytes` `tez` `bool` `key_hash` `date` `duration` and `address`.

Sets are built with `[ ; ]`:

```javascript
variable s : set<int> = [ 1; 3; 5; 12 ] 
```

The `set_contains` operator returns true if an element is an element of the set, false otherwise.

```javascript
effect {
  var s : set<int> = [0; 1 ; 2; 3];
  var t : bool     = set_contains(s, 2); /* true */
}
```

The `set_add` operator returns a set with an extra element if the element is not already in the argument set. It returns the same set if the element is already in the set. 

```javascript
effect {
  var s : set<int> = [0; 1 ; 2; 3];
  var s1 = set_add(s, 5); /* [0; 1 ; 2; 3; 5] */
}
```

The `set_remove` operator returns a set with an element removed if the element was present in the argument set . It returns the same set if the element is not in the set.

```javascript
effect {
  var s : set<int> = [0; 1 ; 2; 3];
  var s1 = set_remove(s, 2); /* [0; 1 ; 3] */
}
```

The `set_length` operator returns the number of elements in the set.

```javascript
effect {
  var s : set<int> = [0; 1 ; 2; 3];
  var l = set_length(s); /* 4 */
}
```

## Maps

Maps are collections of pairs \(key, value\). It is possible to retrieve efficiently the value associated to a key.

Key values may currently be only builtin types : `int` `nat` `string` `bytes` `tez` `bool` `key_hash` `date` `duration` and `address`. Values may be any type, except assets.

Maps are built with `[ ( , ) ;  ]`:

```javascript
variable m : map<int,string> = [ (0,"a string"); (1,"another string") ]
```

The `map_put` operator returns a map with a new pair \(key,value\) if the key is not associated in the argument map. It returns the same map otherwise.

```javascript
effect {
  var m : map<int,string> = [ (0,"a string"); (1,"another string") ];
  var m2 = map_put(m, 2, "and another ...");
}
```

The `map_remove` operator returns a map that does not pair the argument key.

```javascript
effect {
  var m : map<int,string> = [ (0,"a string"); (1,"another string") ];
  var m2 = map_remove(m, 0); /* [ (1,"another string") ] */
}
```

The `[ ]`operator returns an option of the value associated to the argument key.

```javascript
effect {
  var m : map<int,string> = [ (0,"a string"); (1,"another string") ];
  var v = m[1]; /* some("another string") */
  var v = m[42]; /* none */
}
```

The `map_contains` operator returns true if the map associates a value to the argument key, false otherwise.

```javascript
effect {
  var m : map<int,string> = [ (0,"a string"); (1,"another string") ];
  var t = map_contains(m, 0);  /* true */
  var v = map_contains(m, 42); /* false */
}
```

The `map_length` operator returns the number of pairs in the map.

```javascript
effect {
  var m : map<int,string> = [ (0,"a string"); (1,"another string") ];
  var l = map_length(m); /* 2 */
}
```

