---
description: >-
  Archetype is a smart contract development solution dedicated to contract
  quality insurance.
---

# What is Archetype?

Archetype is funded by [Tezos](https://tezos.com) and developed by [edukera](https://edukera.com).

## Smart contracts

Smart contracts are programs which are executed on the blockchain. They were introduced in 2015 by [Ethereum](https://www.ethereum.org/). They allow to read and write virtually any data on the blockchain. 

This literally unleashed the full potential of the blockchains because it created the possibility to develop a new class of application which integrate blockchains as a public and distributed database; These application are called Dapp. 

A smart contract is similar to a stored procedure on a public distributed database. As such, they must ensure the **logical consistency and integrity** of the data.

## What's the problem?

In a nutshell, smart contracts break one of the key principles of the blockchain, which is the _trust-less principle_.

What is meant by _trust-less_ is that, with a blockchain, there is no need for a trusted third party to be confident in the fact the blockchain is doing what it is intended to, which is to manage transactions. Indeed blockchains are secure by design:  they are decentralised and immutable; plus every change in the system is validated by a consensus algorithm.

A smart contract is a _standard_ program, and, as such, it does not come with any guarantee that it will function correctly. We all have in mind the TheDAO incident: a bug in the smart contract made it possible to withdraw several times the money you would put in the contract. As a result, more than 100 millions dollars were lost.

It’s a highly non trivial question to decide whether a program is correct or not, even for the programmer. Before deploying a smart contract, current development best practices include a phase of testing. However, testing is limited by the capacity to cover all possible situations of execution. 

## Required services

In order to circumvent the inherent potential inconsistency of the smart contract,  Archetype  has identified several services which enhances the _apriori_ level of confidence you may have in a smart contract, before deploying it.

#### Verify

Program verification is the gold standard when it comes to trust-less software quality.

What is program formal verification?

Say you have a program and a specific property that the program is supposed to have; say this property is written in formal logic. Program verification consists in figuring out a **mathematical proof** that the program has the property.

A mathematical proof is the perfect trust-less guarantee of the program quality because the only thing you need to is to check that the proof is correct towards the property and the program. This  question is known to be _decidable_, which means that it is possible to have a computer check the correctness of the proof automatically.

Are there limits? 

Yes of course. First, the guarantee you get is limited to the property you have been able to identify. A critical property may still be forgotten!

Second, you need to be skilled in formal methods to formalise the contract properties, and even more to prove the properties, especially if the property is complex.

#### Test

When program verification is too complex or too expensive, it is necessary to setup test batteries to provide standard quality insurance.

#### Simulate

Observing a contract in action is a quick way to gain a reasonable level of insight into the behaviour of a smart contract.

#### Document

It is always good to read what the smart contract designer thinks about what it is supposed to do! If it is aligned with the insights provided by program verification and simulation, it will reinforce the confidence you have in the contract.

#### Execute

This last service is obvious, and has to do with the possibility to execute the smart contract on the blockchain... Without this service, the others would not be relevant.

## Solution frameworks

For each of the services identified above, an existing framework has been selected. A compromise between _ease of use_ and _performance_ was the selection criteria. 

This selection is not definitive nor exhaustive. It should be considered as a starting point.

The following table shows the selected solution foreach service and the main benefits from it:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Service</th>
      <th style="text-align:left">Solution</th>
      <th style="text-align:left">Benefits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Verify</td>
      <td style="text-align:left"><a href="http://why3.lri.fr/">Why3</a>
      </td>
      <td style="text-align:left">
        <p>High level of automation.</p>
        <p>Why3 is a verification framework based on the Hoare Logic. It generates
          verification tasks for external SMT solvers (alt-ergo, cvc, Z3, ...). Verification
          tasks may also be exported to <a href="https://coq.inria.fr">coq</a>.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Test</td>
      <td style="text-align:left"><a href="https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf">Quick-check</a>
      </td>
      <td style="text-align:left">Automated random test generation</td>
    </tr>
    <tr>
      <td style="text-align:left">Simulate</td>
      <td style="text-align:left"><a href="https://gsuite.google.com/intl/en_za/products/sheets/">Google spreadsheets</a>
      </td>
      <td style="text-align:left">Cloud based spreadsheet with complete scripting capability</td>
    </tr>
    <tr>
      <td style="text-align:left">Document</td>
      <td style="text-align:left"><a href="https://en.wikipedia.org/wiki/Markdown">Markdown</a>
      </td>
      <td style="text-align:left">Easy to write and read; easy to convert to other formats (HTML, pdf, ...)</td>
    </tr>
    <tr>
      <td style="text-align:left">Execute</td>
      <td style="text-align:left"><a href="http://www.liquidity-lang.org/">Liquidity</a>
      </td>
      <td style="text-align:left">High level language which compiles to <a href="https://tezos.gitlab.io/master/whitedoc/michelson.html">Michelson</a>.</td>
    </tr>
  </tbody>
</table>{% hint style="warning" %}
'Test' and 'Simulate' services will be available in 2020.
{% endhint %}

The purpose of the archetype Test service is not to replace the standard test approach, but rather to enhance it with random test generation. The idea is to leverage formal properties to derive random and relevant tests.

## A single language

Each framework identified in the previous section comes with a specific language:

| Solution | Language | Type of language |
| :--- | :--- | :--- |
| Why3 | Whyml \(\*.mlw\) | ml language, like ocaml,  with specific instructions for verification |
| Google spreadsheets | Google script \(\*.gs\) | Similar to javascript |
| Document | Markdown \(\*.md\) | plain text with minimalist formatting  instructions for maximum readability  \(unlike HTML markup tags\) |
| Liquidity | \*.liq | ml language |

As a consequence, in order to benefit from these framework, you need to develop several versions of the smart contract, one for each framework.

The fondamental issue, beyond time and skills, is the issue of logical consistency between each version. 

How to make sure that the version for formal verification is consistent with the version for execution? Some verification frameworks have their own solution \(namely extraction\), but what about the consistency over the entire set of services?

Hence the need for a ****single language to describe the business logic of an archetype contract, from which the different operational versions may be derived.

![archetype target languages ](.gitbook/assets/arch_targets_2.png)

{% hint style="info" %}
In a nutshell, archetype is a programming language which is transcoded to target languages for specific services.
{% endhint %}

As such, archetype may also serve as a target language for existing language to benefit from archetype services.  

Follow the link below to read more about the archetype language features.

{% page-ref page="archetype-language/" %}

## Contract library

Archetype comes with a library of several dozen of contract archetypes. Its purpose is to identify  typical contracts to illustrate the archetype language, and bootstrap or inspire your development.

The library cover some key blockchain processes :

* purchase transaction \(escrow\)
* decision process \(voting, auction\)
* investment \(tokens, market place, financial notes\)
* insurance
* other frameworks example for comparison \(Hyper ledger, clause.io, ...\)

Some of these contracts come with relevant properties. Feel free to submit any new property you may find relevant.

{% page-ref page="contract-library/escrow/" %}

{% page-ref page="contract-library/auction/" %}

{% page-ref page="contract-library/governance/" %}

{% page-ref page="contract-library/miles/" %}

{% page-ref page="contract-library/finance/" %}

{% page-ref page="contract-library/tokens/" %}

