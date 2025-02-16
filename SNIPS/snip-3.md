---
snip: 3
title: Non-Fungible Token Standard
author: Abdelhamid Bakhta <abdelhamid.bakhta@gmail.com>
status: Final
type: Standards Track
category: SRC
created: 2022-06-03
---

## Simple Summary

A standard interface for non-fungible tokens, also known as deeds.

Inspired by [EIP-721](https://eips.ethereum.org/EIPS/eip-721).

## Abstract

See <https://eips.ethereum.org/EIPS/eip-721#abstract>.

## Motivation

See <https://eips.ethereum.org/EIPS/eip-721#motivation>.

## Specification

### Methods

**NOTES**:

- The following specifications use syntax from Cairo `2.9.2` (or above)

### balanceOf

Count all NFTs assigned to an owner.

NFTs assigned to the zero address are considered invalid, and this function throws for queries about the zero address.

```cairo
fn balanceOf(self: @TState, account: ContractAddress) -> u256
```

### ownerOf

Find the owner of an NFT.

NFTs assigned to zero address are considered invalid, and queries about them do throw.

```cairo
fn ownerOf(self: @TState, tokenId: u256) -> ContractAddress
```

### safeTransferFrom

Transfers the ownership of an NFT from one address to another address.

Throws unless `get_caller_address` is the current owner, an authorized operator, or the approved address for this NFT.

Throws if `from` is not the current owner.

```cairo
fn safeTransferFrom(
    ref self: TState,
    from: ContractAddress,
    to: ContractAddress,
    tokenId: u256,
    data: Span<felt252>
)
```

### transferFrom

```cairo
fn transferFrom(ref self: TState, from: ContractAddress, to: ContractAddress, tokenId: u256)
```

### approve

```cairo
fn approve(ref self: TState, approved: ContractAddress, tokenId: u256)
```

### setApprovalForAll

```cairo
fn setApprovalForAll(ref self: TState, operator: ContractAddress, approved: bool)
```

### getApproved

```cairo
fn getApproved(self: @TState, tokenId: u256) -> ContractAddress
```

### isApprovedForAll

```cairo
fn isApprovedForAll(self: @TState, owner: ContractAddress, operator: ContractAddress) -> bool
```

### Events

#### Transfer

```cairo
/// Emitted when `token_id` token is transferred from `from` to `to`.
#[derive(Drop, PartialEq, starknet::Event)]
struct Transfer {
    #[key]
    from: ContractAddress,
    #[key]
    to: ContractAddress,
    #[key]
    token_id: u256,
}
```

#### Approval

```cairo
/// Emitted when `owner` enables `approved` to manage the `token_id` token.
#[derive(Drop, PartialEq, starknet::Event)]
struct Approval {
    #[key]
    owner: ContractAddress,
    #[key]
    approved: ContractAddress,
    #[key]
    token_id: u256,
}
```

#### ApprovalForAll

```cairo
/// Emitted when `owner` enables or disables (`approved`) `operator` to manage
/// all of its assets.
#[derive(Drop, PartialEq, starknet::Event)]
struct ApprovalForAll {
    #[key]
    owner: ContractAddress,
    #[key]
    operator: ContractAddress,
    approved: bool,
}
```

## Implementation

Example implementations are available at

- [OpenZeppelin implementation](https://github.com/OpenZeppelin/cairo-contracts/blob/main/packages/token/src/erc721/erc721.cairo)

## History

## Copyright

Copyright and related rights waived via [MIT](../LICENSE).
