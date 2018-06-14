# Multi-signature Minting

- Status: proposed
- Type: enhancement
- Related components: `protocol`, `coinstake-split`, `transaction-timestamp-removal`
- Start Date: 22-March-2017
- Discussion: https://github.com/peercoin/rfcs/issues/4
- Author: hrobeers

## Summary
An advantage of splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction, as described in [RFC-0002](../0002-split-coinstake-transaction/0002-split-coinstake-transaction.md), allows the *coin-age consumption* transaction to be pre-signed off-line.
In combination with multi-signature scripts, this could serve as an alternative to *cold minting* by allowing for *air-gapped minting*.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- The verb "TO MINT" or "MINTING" refers to the action of trying to create a Block using Proof-of-Stake. "STAKING" is chosen over "MINTING" to avoid confusion with "MINING".
- The "MINTER" is the node that puts the final signature on a block that is included in the block chain.

## Motivation
Multi-signature minting uses the protocol adjustments made in [RFC-0002](../0002-split-coinstake-transaction/0002-split-coinstake-transaction.md) and [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md) to achieve the goal of separating the block creation process from the stake signing process.
This separation allows coins secured by offline or hardware wallets to participate in the minting process by requiring only one signature of the minting multisignature address at block creation time.
A 2-of-3 multisignature script can be composed using one on-line key used for minting and two off-line keys, so in order to spend the coins one needs access to at least one of the off-line keys.
The multisignature minting process would require the holder to pre-sign the *coin-age consumption* transaction off-line and send it off to his minter to find stake for that output using the minting key.
The on-line minter would be unable to spend the coins in any other way than broadcasting the *coin-age consumption* transaction that returns all funds to the sender, consuming the coin-age and sending the reward back to the minting address.

## Detailed Design
In the current protocol, multisignature minting is not possible because the block signature is required to match the public key of the coinstake transaction.
Adding a simple rule for multisignature scripts that blocks should be signed by just one of it's public keys would enable multi-signature minting.
With implementation of the dependent RFCs, the minter is free to modify the timestamp and the block reward through the coinbase transaction at block creation time while the owner of the coins can sign the coinstake transaction in advance on an isolated machine.

## Advantages

* Enabling multisignature addresses to mint. While the coinstake would need to be signed by the required number of signatures, the block would only need to be signed by any one member of the multi-sig regardless of the number of signatures required for a normal transaction. 
* Allowing users to only access their wallets once every maturation period (30 days in the current protocol) while still minting continuously, instead of having to give continuous wallet access to their clients.
* Allowing mint pools to exist, though this will require third party infrastructure to navigate the periodic minting process.

## Drawbacks

* Mint pools have existential governance issues, similar to mining pools, in which much of the blockchain governance is transfered to the pool rather than the coin holder.  While this proposal is expected to increase minter participation, it is questionable about whether that participation is truly 'distributed'.

## Dependencies

* [RFC-0002](../0002-split-coinstake-transaction/0002-split-coinstake-transaction.md)
* [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md)

## Alternatives

* [Cold Storage Minting Proposals](https://talk.peercoin.net/t/cold-storage-minting-proposal/2336), one of the highlights of which is Sigmike's cold minting proposal in which there would be a special kind of multisignature address with two private keys, one for transacting and one for minting. It is worth noting that these past proposals are compatible with the current proposed changes.

## Synergy with Cold Minting
There are three roles played by the Peercoin wallet: 1. holding the coins, 2. moving the coins, and 3. minting the coins.
*Cold minting*, once developed, would directly allow for the separation of roles 2 and 3.
Multisignature minting can be leveraged to provide further separation of 1 and 3.
We can fill the roles with names: 1. cold wallet, 2. hot wallet, 3. pool operator.

As an example, we will use Sigmike's *mint key* and *move key* proposal and a 2-of-6 address, such that there are 3 *mint keys* and 3 *move keys*.
The cold wallet will consist of 2 *move keys*.
The hot wallet will consist of 1 *move key* and 2 *mint keys*.
The pool operator will be given 1 *mint key*.
The hot wallet fully signs coinstake transactions every 30 days and gives them to the pool operator to build blocks, such that they can sign the coinbase transaction using the third key.
The coins cannot be moved without at least one cold wallet key.
The benefit of this arrangement is that switching pool operators does not require moving the coins or accessing the cold wallet.
Indeed, the *mint key* given to the pool operator has so little power that it can be given out freely to pool operators.

When switching to a new pool without moving the coins, the old pool operator would still have signed coinstake transactions that it can mint.
By providing the new pool operator with signed coinstake transactions from the same outputs, the new pool operator would have the power to cause blocks submitted by the old pool operator to be banned.

This is to be contrasted with a 1-of-2 address with a *mint key* held by a pool operator or hot wallet and a *move key* held in cold storage, which is the quintessential *cold minting* and does not require splitting the coinstake at all.
However, when switching to a different pool operator or when the hot wallet is compromised, the coins must be moved and the coinage burned to prevent the unwanted use of that coinage.
The 2-of-4+ multisignature address with *cold minting* allows for varying degrees of security, depending on how many of the keys are cold or hot (the hot wallet and the pool operator need at least one *mint key* each, and there have to be at least 2 *move keys*).
The driving benefit of a setup like this is the flexibility to change pool operators without accessing cold keys or consuming coinage.
