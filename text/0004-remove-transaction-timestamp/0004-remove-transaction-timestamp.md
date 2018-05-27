# Remove Transaction Timestamp

- Status: accepted
- Type: bugfix
- Related components: `protocol`, `multisignature-minting`
- Start Date: 17-May-2017
- Discussion: https://github.com/peercoin/rfcs/issues/5 , https://talk.peercoin.net/t/rfc-0004-remove-transaction-timestamp/4867/14
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: hrobeers

## Summary

Peercoin's transaction timestamp is not strictly needed by the protocol and can be abitrarily chosen by the transactor with very little restrictions.
The timestamp provides a small but unfair advantage to educated users and breaks compatibility with Bitcoin infrastructure.
Removing the transaction timestamp facilitates broader adoption of the Peercoin blockchain by increasing the amount of compatible tools and enables the development of protocol improvements like [multisignature minting](../0003-multisig-minting/0003-multisig-minting.md).


## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Peercoin's transaction timestamp is used to determine the coinage of an unspent output to calculate it's coinstake hashvalue.
For regular transactions, the timestamp can be abitrarily chosen by the transactor with the only restriction that it should be greater or equal than the highest timestamp of it's inputs.
Therefore, a transactor can happily [transact without burning coinage](#transacting-without-burning-coinage), which is not a security concern but it might provide an unfair advantage to educated users that can optimally group their outputs for finding stake.

Only the coinstake transaction is for obvious reasons forced to equal the block's timestamp, raising the question why a transaction timestamp is needed.
This proposal advocates the removal of the transaction timestamp in favor of making every transaction inherit the block timestamp.
Removing the timestamp solves the unfair advantage to educated users.
But it also makes peercoin's transaction layout compatible with bitcoin, which significalty lowers the threshold to for infrastructure providers to support Peercoin (e.g. block explorers, hardware wallets, exchanges, ...).

### Transacting without burning coinage

The transaction timestamp makes it possible to transact without burning coinage.
One can easily do so by following the steps below:

- Create a raw transaction.
- Determine the highest timestamp of it's inputs
- Replace the raw transaction's timestamp with the highest input timestamp.
- Sign the updated raw transaction.
- Broadcast on the network.

Examples of such transactions can be found on the network.
One such example is transaction `2759d67ec2997f0f920be7b39c88d758246013d6315a44ebd703df9e02b4b295`.
This transaction has a timestamp on the 4th of April 2017, while it is include in a block with timestamp on the 17th of April 2017.
The author of this RFC anti-dated that transaction on purpose as an example of transacting without burning coinage.

## Detailed design

For the removal of the transaction timestamp, the following changes should be considered:

- Transaction signing and validation to be updated to not include the timestamp.
- Update CheckStakeKernelHash to validate against block timestamps of UTXO and coinstake.
- A hard fork incrementing the transaction version.
- ppcoin flag for rawtransaction RPC interface compatibility (whether to include transaction timestamp field)
- JSON RPC methods to determine transaction timestamps based on the block timestamp. (backwards compatibility)

## Drawbacks

Why should we *not* do this?

## Alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions

What parts of the design are still to be done?
