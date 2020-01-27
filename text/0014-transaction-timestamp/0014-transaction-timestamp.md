# Remove Transaction Timestamp

- Status: proposed
- Type: protocol/persistent format change
- Related components: `protocol`, `multisignature-minting`
- Start Date: 27-Jan-2020
- Discussion: https://github.com/peercoin/rfcs/issues/5 , https://talk.peercoin.net/t/rfc-0004-remove-transaction-timestamp/4867/14
- Supersedes: [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md)
- Superseded by:
- Author: sunnyking

## Summary

Transaction timestamp has posed compatibility issues with many of the major Bitcoin infrastructures, such as Lightning Networks.

It has been generally agreed upon that transaction timestamp inside coinstake transactions carries some security purposes, especially in the light of cold minting and minting pools.

This proposal serves as a middle ground that regular transactions (excluding coinbase and coinstake) have the timestamp field removed. This allows the non-broadcast special coinbase and coinstake transactions to keep timestamp and avoid security concerns.

Note this proposal is considered a protocol change requiring mandatory node upgrade.

It has been argued in [remove transaction timestamp](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md) that removing the transaction timestamp facilitates broader adoption of the Peercoin blockchain by increasing the amount of compatible tools. As to the development of protocol improvements like [multisignature minting](../0003-multisig-minting/0003-multisig-minting.md), it should be able to accommodate the continued existence of coinstake timestamp field.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Consideration

Regular transactions' timestamp is used to determine the coinage of an unspent output. Once removed, it is understood that the transaction timestamp is implicitly the same as the block timestamp of the block that contains it.

However, this interpretation must consider the fact that the transaction can exist on multiple blocks of a given node's blocktree, albeit only on no more than one block of a particular blockchain, such as the main chain of the given node.

Thus, the calculation of coinage is now dependent on the particular main chain of the given node. This is not a concern because the minting protocol assumes consensus has been reached within about 20 days. The transaction output is not allowed to mint until 30-days later, by which time it is presumed that the network has long reached consensus about which block is the main chain block containing the said transaction.

Infrastructures dealing with entire blockchain histories such as explorers and wallets still need to support previous generation regular transaction format that is incompatible with Bitcoin. Infrastruture dealing only with current transactions such as new exchanges, Lightning Network, transaction and smart contract tools might be fully compatible with Bitcoin transactions.

## Detailed design

The following sections are adapted from [remove transaction timestamp](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md).

For the removal of the regular transaction timestamp, the following changes should be considered:

- Transaction signing and validation to be updated to not include the timestamp.
- Update CheckStakeKernelHash to validate against block timestamps of UTXO and coinstake.
- A hard fork incrementing the transaction version.
- ppcoin flag for rawtransaction RPC interface compatibility (whether to include transaction timestamp field)
- JSON RPC methods to determine transaction timestamps based on the block timestamp. (backwards compatibility)

## Advantages

* Prevents transacting without burning coinage.
* Third party tools from other cryptocurrency projects are easier to adapt.
* Allows users to not include timestamps in their transactions, freeing up space or lowering the fee users must pay.

## Drawbacks

* Hardfork of protocol rules to refer to the block timestamp instead of the transaction timestamp in all instances.
* Full nodes will need to support both timestamp conventions in order to parse both new and historical transaction IDs.

## Alternatives

