---
description: (from version 1.2.1)
---

# Metadata \(TZIP-16\)

Archetype provides the possibility to add metadata to the contract according to the TZIP-16 specification:

{% embed url="https://gitlab.com/tzip/tzip/-/blob/master/proposals/tzip-16/tzip-16.md" %}

In a nutshell, metadata is stored in a big map named `%metadata` with an emtpy entry associated to the information where to find the metadata: either in this same big map or in an external resource \(other contract, off-chain server\).

Archetype provides the possiblity either to store the metadata in the `%metadata` bigmap or on an off-chain server. It may be done at compilation time, or from the contract code.

The example below shows how to add the following metadata:

{% code title="simple.json" %}
```javascript
{
  "name": "simple",
  "description": "Just a simple metadata test",
  "version": "0.1",
  "license": "MIT",
  "authors": [
    "Archetype team <archetype-dev@edukera.com>"
  ],
  "homepage": "https://archetype-lang.org/"
}
```
{% endcode %}

to the dummy contract:

{% code title="simple.arl" %}
```javascript
archetype simple

variable n : nat = 0

entry exec () { n := 42; }
```
{% endcode %}

## At compilation

### In storage

The following command adds the metadata in the contract storage:

```javascript
$ ./archetype.exe --metadata-storage simple.json -c simple.arl 
# (Pair 0 { Elt "" 0x74657a6f732d73746f726167653a68657265; Elt "here" 0x7b0a2020226e616d65223a202273696d706c65222c0a2020226465736372697074696f6e223a20224a75737420612073696d706c652074657374222c0a20202276657273696f6e223a2022302e31222c0a2020226c6963656e7365223a20224d4954222c0a202022617574686f7273223a205b0a2020202022417263686574797065207465616d203c6172636865747970652d646576406564756b6572612e636f6d3e220a20205d2c0a202022686f6d6570616765223a202268747470733a2f2f7777772e6564756b6572612e636f6d222c0a2020227669657773223a205b0a202020207b0a202020202020226e616d65223a20226765742d76616c7565222c0a202020202020226465736372697074696f6e223a2022476574207468652063757272656e742076616c7565206f662074686520636f6e74726163742e222c0a2020202020202270757265223a20747275652c0a20202020202022696d706c656d656e746174696f6e73223a205b0a20202020202020207b0a20202020202020202020226d696368656c736f6e2d73746f726167652d76696577223a207b0a2020202020202020202020202272657475726e2d74797065223a207b0a2020202020202020202020202020227072696d223a20226e6174220a2020202020202020202020207d2c0a20202020202020202020202022636f6465223a205b5d2c0a20202020202020202020202022616e6e6f746174696f6e73223a205b5d0a202020202020202020207d0a20202020202020207d0a2020202020205d0a202020207d0a20205d0a7d0a })
{
  storage (pair (nat %n) (big_map %metadata string bytes));
  parameter unit;
  code { UNPAIR;
         DIP { UNPAIR; SWAP };
         DROP;
         PUSH nat 42;
         DIP { DIG 1; DROP };
         DUG 1;
         SWAP;
         PAIR;
         NIL operation;
         PAIR };
}
```

We see line 4 that the `%metadata` big-map is added to the storage.

The Archetype compiler generates the initial storage value to use when deploying the contract.

### In URI

The following command enables to add the URL of the metadata simple.json file. 

```javascript
$ ./archetype.exe --metadata-uri "http://archetype-lang.org/simple.json" -c simple.arl 
# (Pair 0 { Elt "" 0x687474703a2f2f6172636865747970652d6c616e672e6f72672f73696d706c652e6a736f6e })
{
  storage (pair (nat %n) (big_map %metadata string bytes));
  parameter unit;
  code { UNPAIR;
         DIP { UNPAIR; SWAP };
         DROP;
         PUSH nat 42;
         DIP { DIG 1; DROP };
         DUG 1;
         SWAP;
         PAIR;
         NIL operation;
         PAIR };
}
```

## From contract code

`metadata` is a builtin element of the language. It may be accessed without the need for declaration:

```javascript
entry setmetadata (m : bytes) {
  metadata := put(metadata, "here", m);
}
getter getmetadatauri () {
  return metadata[""];
}
```

