# Multi-signature Minting

- Status: proposed
- Type: enhancement
- Related components: `protocol`, `coinstake-split`, `transaction-timestamp-removal`
- Start Date: 22-March-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Author: hrobeers

## Summary
An advantage of splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction, as described in [RFC-0002](../0002-split-coinstake-transaction/0002-split-coinstake-transaction.md), allows the *coin-age consumption* transaction to be pre-signed off-line.
In combination with mutli-signature scripts, this could serve as an alternative to *cold minting*.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- The verb "TO MINT" or "MINTING" refers to the action of trying to create a Block using Proof-of-Stake. "STAKING" is chosen over "MINTING" to avoid confusion with "MINING".

## Motivation
Multi-signature minting requires minimal changes to the protocol, provided that [RFC-0002](../0002-split-coinstake-transaction/0002-split-coinstake-transaction.md) and [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md) are implemented, and allows coins secured by offline or hardware wallets to actively participate in the minting process.
A 2-of-3 multisignature script can be composed using one mint key and two off-line keys, so in order to spend the coins one at least needs access to one of the off-line keys.
The multi-signature minting process would require the holder to pre-sign the *coin-age consumption* transaction off-line and send it off to his minter to find stake for that output using the minting key.
The on-line minter would be unable to spend the coins in any other way than broadcasting the *coin-age consumption* transaction that returns all funds to sender only consuming the coin-age and a transaction fee.
A dependency on [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md) exists to allow pre-signing of the *coin-age consumption* transaction.

## Detailed Design
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction requires only the *coin-age consumption* transaction to be fully signed by the minter.
Since the block reward amount is moved to the *monetary creation* transaction, taking the place of the currently unused coinbase transaction, the block reward is excluded from the *coin-age consumption* signature.
Therefore, the minter can modify the coinbase timestamp and block reward without having to update the *coin-age consumption* signature, allowing him to find stake for an externally signed *coin-age consumption* transaction.

Multi-signature minting is not possible in the current protocol because the block signature is required to match the public key of the coinstake transaction.
Adding a simple rule for multi-signature scripts that blocks should be signed by one of it's public keys would enable multi-signature minting.

## Advantages

* Enabling multiple-signature addresses to mint. While the coinstake would need to be signed by the required number of signatures, the block would only need to be signed by any one member of the multi-sig regardless of the number of signatures required for a normal transaction. 
* Allowing users to only access their wallets once every maturation period (30 days in the current protocol) while still minting continuously, instead of having to give continuous wallet access to their clients.
* Allowing mint pools to exist, though this will require third party infrastructure to navigate the periodic minting process.

## Drawbacks

* Dependency on other RFCs with drawbacks.
* Mint pools have existential governance issues, similar to mining pools, in which much of the blockchain governance is transfered to the pool rather than the coin holder.  While this proposal is expected to increase minter participation, it is questionable about whether that participation is truly 'distributed'.

## Alternatives

* [Cold Storage Proposals](https://talk.peercoin.net/t/cold-storage-minting-proposal/2336), one of the highlights of which is Sigmike's multi-sig proposal in which there would be a special kind of multi-sig address with two private keys, one for transacting and one for minting. It is worth noting that these past proposals are compatible with the current proposed changes.
