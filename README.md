# Arbitrum ERC20 Bridge Review
<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/9760967e-c295-43af-8da7-13acb3bf1890" />

This repository contains my review notes on the Arbitrum ERC20 bridge paths.

## What This Review Covers

Arbitrum moves ERC20 assets between L1 and L2 through a router/gateway model:

- on deposit, ERC20 tokens are escrowed on L1 and credited on L2
- on withdrawal, the L2 token representation is burned and the L1 escrowed asset is released

This review is focused on the ERC20 deposit and withdrawal paths at the bridge contract / application layer.

## Scope

My current scope here is:

- L1 router and L1 gateway deposit logic
- L2 gateway deposit finalization logic
- L2 gateway withdrawal initiation logic
- L1 gateway withdrawal finalization logic
- surrounding config, routing, init, and upgrade surface

My current scope here is focused on the bridge application layer rather than a deeper transport-layer review of Nitro internals. Because of that, components such as Inbox, Outbox, Bridge, and other message-delivery / execution internals are treated here only as transport boundaries between the reviewed L1 and L2 contract-layer paths.

## Review Structure

- [deposit-review.md](deposit-review.md)
  ERC20 deposit path review from L1 routing and escrow to L2 final credit.

- [withdraw-review.md](withdraw-review.md)
  ERC20 withdrawal path review from L2 burn and message creation to L1 final release.

- [out-of-flow-review.md](out-of-flow-review.md)
  Review of the surrounding config, routing mutation, init, and upgrade surface.

- [global-review.md](global-review.md)
  System-level guarantees that connect deposit, withdrawal, and the surrounding admin/config layer.
