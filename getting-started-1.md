# Getting started

## Installation

Installation requires [opam](https://opam.ocaml.org/). Please refer to this [page](https://opam.ocaml.org/doc/Install.html) for opam installation instructions.

Once opam installed:

```
$ opam install archetype
```

{% hint style="info" %}
 current archetype version: **1.1.2**
{% endhint %}

## VS code extension installation

Please refer to this [page](https://code.visualstudio.com/download) to download and install VS code.

Install through VS Code extensions. Search for `archetype`:

[Visual Studio Code Marketplace: archetype](https://marketplace.visualstudio.com/items?itemName=edukera.archetype)

Can also be installed with VS Code Quick Open: press `Cmd/Ctrl + P`, paste the following command, and press enter.

```text
ext install edukera.archetype
```

The vscode extension is configured via the normal vscode settings screen.

{% hint style="warning" %}
Under Mac OS, make sure that VS Code inherits from opam's environement variables, so that `archetype` command is known.  

For example, insert `eval $(opam env)` in the shell profile file \(`.bash_profile` or `.zshrc`\) and launch VS code from Terminal with `open -a Visual \Studio \Code` \(or an alias\). 
{% endhint %}

## Verification tools

Archetype generates Why3 file format \(mlw\) for contract verification purpose. 

### Why3

The current version of why3 is **1.3.3**. It is installed with opam:

```text
$ opam install why3
```

You also need to install the Why3 IDE \(interface\) \(currently 1.3.3\):

```text
$ opam install why3-ide
```

The IDE uses GTK library that may need to be installed with the proper system-dependent package management system \(`apt` for linux, `brew` for mac os, ...\).

{% hint style="info" %}
Visit the official [why3](http://why3.lri.fr/) website for more information.
{% endhint %}

### Solvers

Why3 itself generates contracts' proof obligations and transcodes them to several SMT solvers languages. 

Each solver needs to be installed separately. Once the solvers are installed \(see below\), they need to be detected by Why3 with the following command:

```text
$ why3 config --detect-provers
```

#### Alt-ergo

Alt-ergo is developed by LRI, where Why3 also comes from. The currently supported version is **2.3.1**.

```text
$ opam install alt-ergo.2.3.1
```

#### CVC4

CVC4 is developed at Stanford. The currently supported version is **1.7**. Download the binary at the following page: 

{% embed url="https://cvc4.github.io/downloads.html" %}

#### Z3

Z3 is developped by Microsoft. The currently supported version is **4.7.1**. Download the binary at the following page:

{% embed url="https://github.com/Z3Prover/z3/releases/tag/z3-4.7.1" %}











 

