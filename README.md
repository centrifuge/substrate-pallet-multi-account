# Substrate Multi Account Pallet

[![License: LGPL v3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
[![Build Status](https://travis-ci.com/centrifuge/substrate-multi-account-pallet.svg?branch=master)](https://travis-ci.com/centrifuge/substrate-multi-account-pallet)

A pallet/runtime module for [Substrate](https://github.com/paritytech/substrate) with multisig functionality with a static `AccountId` and a dynamic `threshold` and set of `signatories`.

## Overview

This module contains functionality to create, update and remove multi accounts, identified by a deterministically generated account ID, as well as (potentially) stateful multisig dispatches.

This module is closely implemented after the multisig functionality of [Substrate's utility pallet](https://github.com/paritytech/substrate/blob/master/frame/utility/src/lib.rs). The main difference is that this module stores static account ids on chain which are not changing when the threshold or signatories are updated.

This module allows multiple signed origins (accounts) to coordinate and dispatch calls from a previously created multi account origin. When dispatching calls, the threshold defined in the multi account defines the number of accounts from the multi account signatory set that must approve it. In the case that the threshold is just one then this is a stateless operation. This is useful for multisig wallets where cryptographic threshold signatures are not available or desired or where a dynamic set of signatories with a static account ID is desired.

The motivation for this module is that a deterministically created account ID from threshold and a set of
signatories as in the utility pallet has certain downsides:

1. When using multisigs for token holdings and changing signatories or the threshold, the tokens need to be transferred to a new account ID, which might be a taxable event, depending on jurisdiction. Even if it's not, it complicates communication with accountants/tax advisors.
2. When using multisigs for staking and changing signatories or the threshold, the account needs to unbond, transfer tokens and re-stake again. That can lead to months of lost staking rewards, depending on the unbonding time.
3. When using multisigs for voting and changing signatories or the threshold, the accounts needs to unvotes, transfer tokens and vote again, which is cumbersome.

The downside of using this multi account module is that it is slightly more complex than the utility multisigs and that it introduces additional state and the requirement of creating multi accounts before using multisigs.

## Interface

### Dispatchable Functions

#### For multi account management

* `create` - Create a new multi account with a threshold and given signatories.
* `update` - Update an existing multi account with a new threshold and new signatories.
* `remove` - Remove an existing multi account.

#### For multisig operations/dispatch

* `call` - Approve and if possible dispatch a call from a multi account origin.
* `approve` - Approve a call from a multi account origin.
* `cancel` - Cancel a call from a multi account origin.

## Notes

Multi account thresholds and signatories can only be updated and removed if all active/ongoing multisig operations have been cancelled. To prevent an outgoing multi account member to block updates/removals, the multi account itself can cancel any active multisig.
