---
snip: 6
title: Standard Account Interface
authors: Martín Triay <martriay@gmail.com>, Julien Niset <julien@argent.xyz>, Eric Nordelo <eric.nordelo39@gmail.com>, Sergio Garcia <sergio@argent.xyz>, Yoav Gaziel <yoav.gaziel@braavos.app>
discussions-to: https://community.starknet.io/t/snip-starknet-standard-account/95665
status: Review
type: Standards Track
category: SRC
created: 2023-07-06
---

## Simple Summary

A standard interface for accounts.

## Abstract

The following standard defines a standard API for smart contracts acting as accounts in Starknet. It provides basic functionality for sending transactions through the contract, as well as functionality for validating signatures supporting interoperability among accounts, protocols, and dapps.


## Motivation

With native Account Abstraction, Starknet has a lot of flexibility in the management of accounts rather than having their behavior determined at the protocol level. Different use cases are and will continue to bring different implementations of accounts to the ecosystem.

Having a Standard Account Interface supports different dapps, protocols, and standard contracts (like tokens) that often require predictable interactions with accounts, either by recognizing them or by expecting certain behaviors that may not be implemented otherwise.

## Specification

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

### Interfaces

**Every SNIP-6 compliant account MUST implement the `SRC6` and `SRC5` (from [SNIP-5](./snip-5.md)) interfaces**, and publish both interface ids through `supports_interface`:

**The interface ID for an account is hardcoded to be `0x2ceccef7f994940b3962a6c67e0ba4fcd37df7d131417c604f91e03caecc1cd`** which matches the original account interface. Note that some methods have changed since the first version, but the interface ID will not change to maintain compatibility.

#### Current interface

```cairo
/// @title Represents a call to a target contract
/// @param to The target contract address
/// @param selector The target function selector
/// @param calldata The serialized function parameters
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Span<felt252>
}

/// @title SRC-6 Standard Account
trait ISRC6 {
    /// @notice Execute a transaction through the account
    /// @param calls The list of calls to execute
    fn __execute__(calls: Array<Call>);

    /// @notice Assert whether the transaction is valid to be executed
    /// @param calls The list of calls to execute
    /// @return The string 'VALID' represented as felt when is valid
    fn __validate__(calls: Array<Call>) -> felt252;

    /// @notice Assert whether a given signature for a given hash is valid
    /// @param hash The hash of the data
    /// @param signature The signature to validate
    /// @return The string 'VALID' represented as felt when the signature is valid
    fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}

/// @title SRC-5 Standard Interface Detection
trait ISRC5 {
    /// @notice Query if a contract implements an interface
    /// @param interface_id The interface identifier, as specified in SRC-5
    /// @return `true` if the contract implements `interface_id`, `false` otherwise
    fn supports_interface(interface_id: felt252) -> bool;
}
```



#### Legacy interface used to calculate the [SRC5](./snip-5.md) interface ID

```cairo
/// @title Represents a call to a target contract
/// @param to The target contract address
/// @param selector The target function selector
/// @param calldata The serialized function parameters
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

/// @title SRC-6 Standard Account
trait ISRC6 {
    /// @notice Execute a transaction through the account
    /// @param calls The list of calls to execute
    /// @return The list of each call's serialized return value
    fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;

    /// @notice Assert whether the transaction is valid to be executed
    /// @param calls The list of calls to execute
    /// @return The string 'VALID' is represented as felt when is valid
    fn __validate__(calls: Array<Call>) -> felt252;

    /// @notice Assert whether a given signature for a given hash is valid
    /// @param hash The hash of the data
    /// @param signature The signature to validate
    /// @return The string 'VALID' is represented as felt when the signature is valid
    fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}
```

Notice that, if the signature is valid, the return value for `is_valid_signature` MUST be the short string literal `VALID`.

## Security

To guarantee the signature cannot be replayed in other accounts or other chains, the data hashed must be unique to the account and the chain.
This is true for starknet transaction signatures and SNIP-12 signatures. However, it's worth noting that this may not necessarily hold for other types of signatures.
Wallets are advised not to sign any data unless they know that it includes the account address and the chain ID

## Rationale

This SNIP aims to standardize the interface for accounts on Starknet. While some functions are required by Starknet itself for an account to function, this standard includes additional functionality that enhances an account's interoperability with the broader Starknet ecosystem.
For example, the `is_valid_signature` function provides a standard way for dapps and protocols to verify signatures, making the account compatible with any dapp or protocol that expects this functionality. Adhering to the SNIP-5 also makes an account more easily discoverable and integrable.
While an account on Starknet could function without adhering to this standard, doing so provides significant benefits in terms of interoperability, discoverability, and user experience. Accounts compliant with SNIP-6 can expect better compatibility and integration with the rest of the Starknet ecosystem.
At the same time, the standard is designed to be minimal and flexible, allowing for a wide variety of account implementations.

## Backwards Compatibility

As mentioned before the interface ID for an account remains `0x2ceccef7f994940b3962a6c67e0ba4fcd37df7d131417c604f91e03caecc1cd`** which matches the original account interface. 

Note that the the current interface id and the legacy one are compatible in the way they can be called.

The latest version of the `__execute__` method returns no data. But older versions used to return the result of the calls. Third parties shouldn't rely on the data being returned.

Currently, multiple accounts are using `bool` as the `is_valid_signature` return value. While in the future we expect that most of the accounts will migrate to this standard, in the meantime, we recommend dapps and protocols using this feature to check for both `true` or `'VALID'`.

## Copyright

Copyright and related rights waived via [MIT](../LICENSE).
