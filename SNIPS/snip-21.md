---
snip: 21
title: Out-of-band fee-on-transfer tokens
author: Moody Salem <moody@ekubo.org>
status: Living
type: SRC
created: 2024-07-29
---

## Simple Summary

Defines a standard for fee-on-transfer tokens that maintains the semantics of fungible token transfers and has no
changes to the existing tokens interface.

## Abstract

Fee-on-transfer is the idea of charging a fee whenever fungible tokens are transferred from one address to another. Such
tokens are very popular on other networks. These tokens have inconsistent implementations and do not work well with many
DeFi protocols, but are still popular for the deflationary aspect.

In particular, some DeFi protocols such as Ekubo Protocol do not require tokens to be transferred to swap or add
liquidity. This allows liquidity providers and swappers to entirely circumvent any possible implementation of a fee
taken on transfer--the token contract does not need to be called at all in order to interact with Ekubo pools.

## Motivation

Due to the demand for this functionality, it's important to define a mechanism that makes the best use of Starknet's
capabilities. This SRC defines an implementation of fee-on-transfer tokens that maintains the semantics of fungible
token transfers, allowing it to be broadly compatible with Starknet DeFi, and also requires no changes to the token
interface. This is possible because of Starknet's native account abstraction and multicall.

## Specification

```cairo
#[starknet::interface]
pub trait IFeeOnTransferToken<TContractState> {
    // Gets the amount of fees already paid for the given sender
    fn get_fees_paid(self: @TContractState, sender: ContractAddress) -> u128;

    // Returns the amount of fees required to transfer the specified amount of tokens from the given sender to the receiver.
    fn compute_fees_required(
        self: @TContractState, sender: ContractAddress, receiver: ContractAddress, amount: u128
    ) -> u128;

    // Pays fees from the given address for the specified sender.
    // If `from` is the caller, then it pays from the caller's balance.
    // Otherwise, it pays from the allowance of the `from` address to the caller as the spender.
    // Returns the total amount of fees paid for the sender
    fn pay_fees_verbose(
        ref self: TContractState, from: ContractAddress, sender: ContractAddress, amount: u128
    ) -> u128;
    
    // Same as pay_fees_verbose but always pays `from` the caller address
    fn pay_fees_for_sender(ref self: TContractState, sender: ContractAddress, amount: u128) -> u128;
    
    // Same as pay_fees_for_sender, but the sender is always the caller address
    fn pay_fees(ref self: TContractState, amount: u128) -> u128;

    // Withdraws any fees paid for the given sender to the specified recipient address.
    // This can be called  by anyone for any address--fees are a transient payment location to enable transfers for a given address. Leftover fees should always be withdrawn in the same transaction.
    // Returns the amount of fees that were withdrawn.
    fn withdraw_fees_paid_verbose(
        ref self: TContractState, sender: ContractAddress, recipient: ContractAddress
    ) -> u128;
    
    // Same as `withdraw_fees_paid_verbose` but recipient is always the caller
    fn withdraw_fees_paid_for_sender(ref self: TContractState, sender: ContractAddress) -> u128;
    
    // Same as `withdraw_fees_paid_for_sender` but the sender and recipient are both set to the caller
    fn withdraw_fees_paid(ref self: TContractState) -> u128;
}
```

The expected usage flow is as follows:

- DApp stores off-chain metadata about whether a token is fee-on-transfer, and implements the following logic if it is
- When depositing this token into a dapp, the `recipient` is the Dapp contract:
    - Dapp calls `compute_fees_required(user, dapp_contract, amount)` off-chain
    - Dapp adds `pay_fees(computed_fees)` call to the list of calls before all dapp calls
    - Dapp adds rest of calls as normal
    - Dapp adds `withdraw_fees_paid()` call to return any overpaid transfer fees
- When withdrawing this token from a dapp, the `receiver` is the caller and the `sender` is the dapp
    - Dapp calls `pay_fees_for_sender(dapp_contract, amount)`
    - Dapp adds rest of calls as normal
    - Dapp adds `withdraw_fees_paid_for_sender(dapp_contract)`

There are other ways this standard may be used, such as handling fees off-chain in a peripheral contract. This design
expects there are no intermediate transfers of the token between the dapp contract and the user, i.e. it relies on
`approve` and `transferFrom` in cases where the called contract is not where the tokens are custodied.

## Implementation

TBD

## History

- Created 2024-07-29

## Copyright

Copyright and related rights waived via [MIT](../LICENSE).
