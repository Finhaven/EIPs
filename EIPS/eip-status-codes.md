---
eip: 000
title: Status Codes
author: Brooklyn Zelenka (@expede) <brooklyn@finhaven.com>, Tom Carchrae (@carchrae) <tom@finhaven.com>, Gleb Naumenko (@naumenkogs) <gleb@finhaven.com>
status: Draft
type: Standards Track
category: ERC
created: 2018-05-04
---

## Simple Summary

Broadly applicable status codes for Ethereum smart contracts.

## Abstract

This standard outlines a set of Ethereum status codes (ESC) in the same vein
as HTTP statuses. This provides a shared contextual language to allow smart contracts
to react to situations autonomously, expose localized error messages to users, and so on.

The current state of the art is to either `revert` and require human intervention,
or return a Boolean pass/fail status. Status codes are similar to reverting with a reason,
and events, but are aimed at on-chain automation and translation.

While clearly related, status codes are orthogonal to "`revert` with reason".
ESCs are not limited to rolling back the transaction, and may represent
known error states without halting execution. They may also represt off-chain conditions,
supply a string to `revert`, signal time delays, and more.

Much like HTTP codes, having a standard set of known codes has many benefits for developers.
They remove friction from needing to develop your own schemes for every contract,
makes inter-contract automation easier, and makes it easier to broadly understand which of the
finite states your request produced. Importantly, it makes it much easier to distinguish
between expected errors states, and truly exceptional conditions that require
halting execution and rolling back state.

## Motivation

### Autonomy

Smart contracts are largely intended to be autonomous. While each contract may
define a specific interface, having a common set of semantic codes can help
developers write code that can react appropriately to various situations.

### Semantic Density

HTTP status codes are widely used for this purpose. BEAM languages use atoms
and tagged tuples to signify much the same information. Both provide a lot of
information both to the programmer (debugging for instance), and to the program
that needs to decide what to do next.

ESCs convey a much richer set of information than Booleans,
and are able to be reacted to autonomously unlike arbitrary strings.

### User Feedback

Since status codes are finite and known in advance, we can provide global,
human-readable sets of status messages. These may also be translated into any language,
differing levels of technical detail, added as `revert` messages, natspecs, and so on.

We also see a desire for this [in transactions](http://eips.ethereum.org/EIPS/eip-658),
and there's no reason that ESCs couldn't be used by the EVM itself.

## Specification

### Format

Codes are returned as the first value of potentially multiple return values.

```solidity
// Status only

function isInt(uint num) public pure returns (byte status) {
    return hex"01";
}

// Status and value

uint8 private counter;

function safeIncrement(uint8 interval) public returns (byte status, uint8 newCounter) {
    uint8 updated = counter + interval;

    if (updated >= counter) {
        counter = updated;
        return (hex"01", updated);
    } else {
        return (hex"00", counter);
    }
}
```

In the rare case that there a multiple codes required to express an idea,
they should be organized in asending order.

### Code Table

| X. Low Nibble                     | 0. Generic              | 10. Permission                | 20. Find/Match/&c           | 30. Negotiation / Offers         | 40. Availability                 | 50. | 60. | 70. | 80. | 90. | A0. | B0. | C0. | D0. | E0. Cryptography & Tokens           | F0. Off Chain                                     |
|-----------------------------------|-------------------------|-------------------------------|-----------------------------|----------------------------------|----------------------------------|-----|-----|-----|-----|-----|-----|-----|-----|-----|-------------------------------------|---------------------------------------------------|
| 0. Failure                        | 0x00 Failure            | 0x10 Disallowed               | 0x20 Not Found              | 0x30 Other Party Disagreed       | 0x40 Unavailable                 |     |     |     |     |     |     |     |     |     | 0xE0 Decrypt Failure                | 0xF0 Off Chain Failure                            |
| 1. Success                        | 0x01 Success            | 0x11 Allowed                  | 0x21 Found                  | 0x31 Other Party Agreed          | 0x41 Available                   |     |     |     |     |     |     |     |     |     | 0xE1 Decrypt Success                | 0xF1 Off Chain Success                            |
| 2. Accepted / Started             | 0x02 Accepted / Started | 0x12 Requested Permission     | 0x22 Match Request Sent     | 0x32 Sent Offer                  | 0x42 You May Begin               |     |     |     |     |     |     |     |     |     | 0xE2 Signed                         | 0xF2 Off Chain Process Started                    |
| 3. Awaiting / Before              | 0x03 Awaiting / Before  | 0x13 Awaiting Permission      | 0x23 Awaiting Match         | 0x33 Awaiting Their Ratification | 0x43 Not Yet Available           |     |     |     |     |     |     |     |     |     | 0xE3 Other Party Signature Required | 0xF3 Awaiting Off Chain Completion                |
| 4. Action Required                | 0x04 Action Required    | 0x14 Awaiting Your Permission | 0x24 Match Request Received | 0x34 Awaiting Your Ratification  | 0x44 Awaiting Your Availability* |     |     |     |     |     |     |     |     |     | 0xE4 Your Signature Required        | 0xF4 Off Chain Action Required                    |
| 5. Expired                        | 0x05 Expired            | 0x15 No Longer Allowed        | 0x25 Out of Range           | 0x35 Offer Expired               | 0x45 No Longer Available         |     |     |     |     |     |     |     |     |     | 0xE5 Token Expired                  | 0xF5 Off Chain Service Unavailable                |
| 6.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| 7.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| 8.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| 9.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| A.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| B.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| C.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| D.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| E.                                |                         |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     |                                                   |
| F. Meta/Info                      | 0x0F Metadata Only      |                               |                             |                                  |                                  |     |     |     |     |     |     |     |     |     |                                     | 0xFF Data Source is Off Chain (ie: no guarantees) |

* Unused regions are available for further extension or custom codes
* You may need to scroll the tables horizontally (they're pretty wide)

### Example Function Change

```solidity
uint256 private startTime;
mapping(address => uint) private counters;

// Before
function increase() public returns (bool _available) {
    if (now < startTime && counters[msg.sender] == 0) {
        return false;
    };

    counters[msg.sender] += 1;
    return true;
}

// After
function increase() pubilic returns (byte _status) {
    if (now < start) { return hex"43"; } // Not yet available
    if (counters[msg.sender] == 0) { return hex"10"; } // Not authorized

    counters[msg.sender] += 1;
    return hex"01"; // Success
}
```

### Example Sequence Diagrams

```
0x03 = Waiting
0x31 = Other Party (ie: not you) Agreed
0x41 = Available
0x43 = Not Yet Available


                          Exchange


AwesomeCoin                 DEX                     TraderBot
     +                       +                          +
     |                       |       buy(AwesomeCoin)   |
     |                       | <------------------------+
     |         buy()         |                          |
     | <---------------------+                          |
     |                       |                          |
     |     Status [0x43]     |                          |
     +---------------------> |   Status [0x03, 0x43]    |
     |                       +------------------------> |
     |                       |                          |
     |                       |        isDoneYet()       |
     |                       | <------------------------+
     |                       |                          |
     |                       |    Status [0x03, 0x43]   |
     |                       +------------------------> |
     |                       |                          |
     |                       |                          |
     |     Status [0x41]     |                          |
     +---------------------> |                          |
     |                       |                          |
     |       buy()           |                          |
     | <---------------------+                          |
     |                       |                          |
     |                       |                          |
     |     Status [0x31]     |                          |
     +---------------------> |      Status [0x31]       |
     |                       +------------------------> |
     |                       |                          |
     |                       |                          |
     |                       |                          |
     |                       |                          |
     +                       +                          +
```



```
0x01 = Generic Success
0x10 = Disallowed
0x11 = Allowed

                                              Token Validation


           Buyer                  RegulatedToken           TokenValidator               IDChecker          SpendLimiter
             +                          +                         +                         +                   +
             |        buy()             |                         |                         |                   |
             +------------------------> |          check()        |                         |                   |
             |                          +-----------------------> |          check()        |                   |
             |                          |                         +-----------------------> |                   |
             |                          |                         |                         |                   |
             |                          |                         |         Status [0x10]   |                   |
             |                          |       Status [0x10]     | <-----------------------+                   |
             |        throw/revert      | <-----------------------+                         |                   |
             | <------------------------+                         |                         |                   |
             |                          |                         |                         |                   |
+---------------------------+           |                         |                         |                   |
|                           |           |                         |                         |                   |
| Updates ID with provider  |           |                         |                         |                   |
|                           |           |                         |                         |                   |
+---------------------------+           |                         |                         |                   |
             |                          |                         |                         |                   |
             |         buy()            |                         |                         |                   |
             +------------------------> |        check()          |                         |                   |
             |                          +-----------------------> |         check()         |                   |
             |                          |                         +-----------------------> |                   |
             |                          |                         |                         |                   |
             |                          |                         |       Status [0x11]     |                   |
             |                          |                         | <-----------------------+                   |
             |                          |                         |                         |                   |
             |                          |                         |                         |   check()         |
             |                          |                         +-------------------------------------------> |
             |                          |                         |                         |                   |
             |                          |                         |                         |  Status [0x11]    |
             |                          |       Status [0x11]     | <-------------------------------------------+
             |        Status [0x01]     | <-----------------------+                         |                   |
             | <------------------------+                         |                         |                   |
             |                          |                         |                         |                   |
             |                          |                         |                         |                   |
             |                          |                         |                         |                   |
             +                          +                         +                         +                   +
```

## Rationale

### Encoding

ESCs are encoded as a `byte`. Hex values break nicely into high and low nibbles:
`category` and `reason`. For instance, `hex"01"` stands for general success
and `hex"00"` for general failure.

`byte` is quite lightweight, and can be easily packed with multiple codes into
a `bytes32` (or similar) if desired. It is also easily interoperable with `uint8`,
cast from `enum`s, and so on.

#### Alternatives

Alternate schemes include `bytes32` and `uint8`. While these work reasonably
well, they have drawbacks.

`uint8` feels even more similar to HTTP status codes, and enums don't require
as much casting. However does not break as evenly as a square table
(256 doesn't look as nice in base 10).

Packing multiple codes into a single `bytes32` is nice in theory, but poses additional
challenges. Unused space may be interpeted as `0x00 Failure`, you can only efficiently
pack four codes at once, and there is a challenge in ensuring that code combinations
are sensible. Forcing four codes into a packed representation encourages multiple
status codes to be returned, which is often more information than strictly nessesary.
This can lead to paradoxical results (ex `0x00` and `0x01` together),
or greater resorces allocated to interpreting 256<sup>4</sup> (4.3 billion) permutations.

### Multiple Returns

While there may be cases where packing a byte array of ESCs may make sense, the simplest,
most forwards-compatible method of transmission is as the first value of a multiple return.

Familiarity is also a motivating factor. A consistent position and encoding together
follow the principle of least surprise. It is both viewable as a "header" in the HTTP analogy,
or like the "tag" in BEAM tagged tupples.

### Human Readable

Developers should not be required to memorize 256 codes. However, they break nicely into a table.
Cognitive load is lowered by organizing the table into categories and reasons.
`0x10` and `0x11` belong to the same category, and `0x04` shares a reason with `0x24`

While this repository includes helper enums, we have found working directly in
the hex values to be quite natural. ESC `0x10` is just as comfortable as HTTP 401, for example.

### Extensiblilty

The `0xA` category is reserved for application-specific statuses.
In the case that 256 codes become insufficient, `bytes1` my be embedded in larger byte arrays.

### EVM Codes

The EVM also returns a status code in transactions; specifically `0x00` and `0x01`.
This proposal both matches the meanings of those two codes, and could later be used
at the EVM level.

### Empty Space

Much like how HTTP status codes have large unused ranges, there are totally empty
sections in this proposal. The intent is to not impose a complete set of codes up front,
and to allow users to suggest uses for these spaces as time progresses.

## Implementation

Reference cases and helper library can be found [here](https://github.com/Finhaven/EthereumStatusCodes)

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
