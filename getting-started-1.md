# Getting started

## Installation

Archetype compiler comes with the [Completium CLI](https://completium.com/docs/cli) :

```text
npm i @completium/completium-cli -g
completium-cli init
```

To know the current version of Archetype:

```text
completium-cli archetype version
```

The current version is `1.2.8`.

## VS code extension installation

Archetype also comes with the Archetype [VS code](https://code.visualstudio.com/download) extension:

[Visual Studio Code Marketplace: archetype](https://marketplace.visualstudio.com/items?itemName=edukera.archetype)

## Verification tools

Archetype generates Why3 file format \(mlw\) for contract verification purpose. 

### Why3

The current version of why3 is **1.4.0**. It is installed with opam:

```bash
opam install why3
```

You also need to install the Why3 IDE \(interface\) \(currently 1.4.0\):

```text
opam install why3-ide
```

The IDE uses GTK library that may need to be installed with the proper system-dependent package management system \(`apt` for linux, `brew` for mac os, ...\).

{% hint style="info" %}
Visit the official [Why3](http://why3.lri.fr/) website for more information.
{% endhint %}

### Solvers

Why3 itself generates contracts' proof obligations and transcodes them to several SMT solvers languages. 

Each solver needs to be installed separately. Once the solvers are installed \(see below\), they need to be detected by Why3 with the following command:

```text
why3 config detect
```

#### Alt-ergo

Alt-ergo is developed by LRI, where Why3 also comes from. The currently supported version is **2.4.0**.

```text
opam install alt-ergo.2.4.0
```

#### CVC4

CVC4 is developed at Stanford. The currently supported version is **1.7**. Download the binary at the following page: 

{% embed url="https://cvc4.github.io/downloads.html" %}

#### Z3

Z3 is developped by Microsoft. The currently supported version is **4.7.1**. Download the binary at the following page:

{% embed url="https://github.com/Z3Prover/z3/releases/tag/z3-4.7.1" %}











 

