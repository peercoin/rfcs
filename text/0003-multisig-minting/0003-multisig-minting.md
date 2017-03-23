# Multi-signature Minting

- Status: proposed
- Type: enhancement
- Related components: `protocol`, `coinstake-split`
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
Multi-signature minting requires minimal changes to the protocol and allows coins secured by offline or hardware wallets to actively participate in the minting process.
A 2-of-3 multisignature script can be composed using one mint key and two off-line keys, so in order to spend the coins one at least needs access to one of the off-line keys.
The multi-signature minting process would require the holder to pre-sign the *coin-age consumption* transaction off-line and send it off to his minter to find stake for that output using the minting key.
The on-line minter would be unable to spend the coins in any other way than broadcasting the *coin-age consumption* transaction that returns all funds to sender only consuming the coin-age and a transaction fee.

## Detailed Design
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction requires only the *coin-age consumption* transaction to be fully signed by the minter.
Because the block reward amount and the creation timestamp are moved to the *monetary creation* transaction, taking the place of the currently unused coinbase transaction, these properties are excluded from the *coin-age consumption* signature.
Therefore, the minter can modify the coinbase timestamp and block reward without having to update the *coin-age consumption* signature, allowing him to find stake for an externally signed *coin-age consumption* transaction.

Multi-signature minting is not possible in the current protocol because the block signature is required to match the public key of the coinstake transaction.
Adding a simple rule for multi-signature scripts that blocks should be signed by one of it's public keys would enable multi-signature minting.

### Example scenarios
TODO

## Advantages

* More secure minting.
* Permanent outsourcing of minting is not possible, every mint needs to be pre-signed by the holder.
* Minimal protocol changes.

## Drawbacks

* Hard fork

## Alternatives

TODO