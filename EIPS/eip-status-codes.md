---
eip: <to be assigned>
title: Status Codes
author: Brooklyn Zelenka (@expede) <brooklyn@finhaven.com>, Tom Carchrae (@carchrae) <tom@finhaven.com>, Gleb Naumenko <gleb@finhaven.com> (@naumenkogs)
discussions-to: <email address>
status: Draft
type: Standards Track
category: ERC
created: 2018-05-04
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->
If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
A short (~200 word) description of the technical issue being addressed.

## Motivation
### Autonomy

Smart contracts are largely intended to be autonomous. While each contract may
define a specific interface, having a common set of semantic codes can help
developers write code that can react appropriately to various situations.

### Semantically Density

HTTP status codes are widely used for this purpose. BEAM languages use atoms
and tagged tuples to signify much the same information. Both provide a lot of
information both to the programmer (debugging for instance), and to the program
that needs to decide what to do next.

ESCs convey a much richer set of information than booleans,
and are able to be reacted to autonomously unlike arbitrary strings.

### User Feedback

Since status codes are finite and known in advance, we can provide global,
human-readable sets of status messages. These may also be translated into any language,
differing levels of technical detail, added as `revert` messages, natspecs, and so on.

We also see a desire for this [in transactions](http://eips.ethereum.org/EIPS/eip-658),
and there's no reason that ESCs couldn't be used by the EVM itself.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->
The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Implementation
<!--The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->
The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.

https://github.com/Finhaven/EthereumStatusCodes

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
