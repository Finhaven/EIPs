## Preamble

    EIP: <to be assigned>
    Title: Token Validation
    Author: Tom Carchrae, Gleb Naumenko, Brooklyn Zelenka
    Type: Standard Track
    Category: ERC
    Status: Draft
    Created: 2018-02-14

## Simple Summary
A protocol for services providing financial transaction validation.

## Abstract
This standard provides a registry contract method for authorizing token transfers.
By nature, this covers both initially issuing tokens to users (ie: transfer from contract to owner),
transferring tokens between users, and token spends.

## Motivation
The tokenization of assets has wide application,
not least of which is financial instruments such as securities.
Most jurisdictions have placed legal constraints on what may be traded,
and who can hold such tokens which are regarded as securities. Broadly this includes KYC and AML validation,
but may also include time-based spend limits, total volume of transactions, and so on.

Regulators and sanctioned third-party compliance agencies need some way to link
off-chain compliance information such as identity and residency to an on-chain service.
The application of this design is broader than legal regulation, encompasing all manner
of business logic permissions for the creation, mangement, and trading of tokens.

## Specification

### TokenValidator

```solidity
interface TokenValidator {
  function check(
    address _token,
    address _user
  ) public returns(uint8 result)

  function check(
    address _token,
    address _from,
    address _to,
    uint256 _amount
  ) public returns (uint8 result)
}
```

#### Methods

##### check/2

`function check(address token, address user) public returns (uint8 resultCode)`

> parameters
> * `token`: the token under review
> * `user`: the user to check
>
> *returns* an 8-bit status code

##### check/4

`function check(address token, address from, address to, uint256 amount) public returns (uint8 resultCode)`

> parameters
> * `token`: the token under review
> * `from`: in the case of a transfer, who is relinquishing token ownership
> * `to`: in the case of a transfer, who is accepting token ownership
> * `amount`: The number of tokens being transferred
>
> *returns* an 8-bit status code

### ValidatedToken

```solidity
interface ValidatedToken {
  event Validation(
    address indexed user,
    uint8   indexed result
  )

  event Validation(
    address indexed from,
    address indexed to,
    uint256 value,
    uint8   indexed result
  )
}
```

#### Events

##### Validation/2

`event Validation(address indexed user, uint8 indexed resultCode)`

This event MUST be fired on return from a call to a `TokenValidator.check/2`.

> parameters
> * `user`: the user to check
> * `resultCode`: a status code


##### Validation/4

```solidity
event Validation(
  address indexed from,
  address indexed to,
  uint256 amount,
  uint8   indexed result
)
```

This event MUST be fired on return from a call to a `TokenValidator.check/4`.

> parameters
> * `from`: in the case of a transfer, who is relinquishing token ownership
> * `to`: in the case of a transfer, who is accepting token ownership
> * `amount`: The number of tokens being transferred
> * `resultCode`: a status code

## Rationale

This proposal includes a financial permissions system on top of any financial token.
This design is not a general roles/permission system. In any system, the more you know
about the context where a function will be called, the more powerful your function can be.
By restricting ourselves to token transfers (ex. ERC20 or EIP-777), we can make
assumptions about the use cases our validators will need to handle, and can make
the API both small, useful, and extensible.

The events are fired by the calling token. Since `Validator`s may aggregate or delagate
to other `Validator`s, it would generate a lot of useless events were it the `Validator`'s responsability.
This is also the reason why we include the `token` in the `call/4` arguments:
a `Validator` cannot rely on `msg.sender` to determine the token that it's about.

We have also seen a similar design from [R-Token](https://github.com/harborhq/r-token) that uses an additional field: `spender`.
While there are potential use cases for this, it's not widely used enough to justify passing
a dummy value along with every call. Instead, such a call would look more like this:

```solidity
function approve(address spender, uint amount) public returns (bool success) {
    if(validator.check(this, msg.sender, spender, amount) == !okStatusCode) {
        return false;
    };

    allowed[msg.sender][spender] = tokens;
    Approval(msg.sender, spender, tokens);
    return true;
}
```

A second `check/2` function is also required, that is mor general-purpose, and does not
specify a transfer amount or recipient. This is intended for general checks,
such as checking roles (admin, owner, &c), or ifa user is on a simple whitelist.
You may still use `check/4`, but passing dummy data just to satisfy a function contract
is generally frowned upon.

We have left the decision to make associated `Validator` addresses public, private, or hardcoded
up to the implementer. The proposed deisgn does not include a centralized registry.
It also does not include an interface for a `Validated` contract.
A token may require one or many `Validator`s for different purposes,
requiring different validations for different, or just a single `Validator`.
The potential use cases are too varied to provide a single unified set of methods.
We have provided a set of example contracts [here](some.link) that may be inherited from for common use cases.

By only defining the opaque validation check, this standard is widely compatible with
ERC-20, EIP-721, EIP-777, future token standards, centralized and decentralized exchanges,
and so on.

## Implementation
[Reference implementations](https://github.com/Finhaven/ValidatedToken/) (WIP)

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
