---
description: Cryptographic utilities.
---

# Crypto

## Addresses

Address literals are prefixed by `@`.

```javascript
variable admin : address = @tz1ZXf37ZNfVupt5GVyrPJ76q8kbjFuD2z7J
variable acontract : address = @KT19NxoYm9rjDQxz2gJiFSY8Z5Mjofq7rebs
```

Addresses may be compared with the 6 comparison operators `= <> < > <= >=`.

## Bytes

Bytes literals are prefixed by `0x`.

```javascript
variable bvalue : bytes = 0x050100000009617263686574797065 // "archetype" in bytes
```

### Expressions

The following operators are available:

* `concat`
* `slice`

```javascript
effect {
    var byt_concat : bytes  = concat(0x12, 0xef);
    var byt_slice : bytes  = slice(0xabcdef01, 1, 2);
}
```

Bytes values are comparable with `= <> < > <= >=` operators.

### Hashing

Hashing functions are available: `blake2b` `sha256` and `sha512`. They convert a byte value to the hashed byte value.

```javascript
var h = sha256(0x050100000009617263686574797065);
```

### pack, unpack

It is possible to convert any value from and to a bytes value with `pack` and `unpack` operators.

The `unpack` operator is parameterized with the expected output type using `<T>` syntax. It returns an value option and hence _does not fail_. 

```javascript
effect {
   var bstr := pack("archetype"); // bstr is a bytes value
   if getopt(unpack<string>(bstr)) = "archetype"
   then transfer 1tz to coder;
}
```

## Keys and Signatures

In smart contracts, it may be necessary to assert the origin of an input data: for example an oracle may need to write a value based on which critical decisions will be made; it is then necessary to make sure that the data written in the contract is delivered by the oracle. The solution is to pass the signed version of the data along with the data.

It is possible to check whether a bytes value has been signed by a given key with the `check_signature` operator. It takes 3 arguments:

1. the public key of the one who supposedly signed the data \(`key` type\)
2. a signed data \(`signature` type\)
3. the bytes data \(`bytes` type\)

It returns true if and only if the _encryption_ by the public key of the bytes data is equal to the signed data.

For example, the following enables an oracle to write a value `outcome` typed `int` in the contract:

```javascript
archetype oraclesetvalue

variable outcome : int option = none

// oracle's public key
constant oracle : key = "edpkurLzuFFL1XyP3fed4u7MsgeywQoQmHM45Bz91PBzDvUjQ9bvdn"

entry setoutcome (packed_outcome : bytes, signed_outcome : signature) {
  effect {
    if check_signature(oracle, signed_outcome, packed_outcome) then (
      outcome := unpack<int>(packed_outcome);
    ) else fail("not signed by oracle");
  }
}
```

Note that anyone can call the `setoutcome` entry. It is ok as long as they possess the data and the data signed by the oracle.

Note also that key literals use the string syntax.

Below is an instance of the above contract with the outcome value set to `42`.

{% embed url="https://you.better-call.dev/carthagenet/KT1AiNKbVi6c74pyxMZJnK4yj2P9GxP2K2d2/operations" %}



