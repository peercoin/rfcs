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
If a *cold minting* protocol is implemented, the conventions proposed could be used to implement secure socialization of pool rewards in a minting pool.

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
* Allowing for socialization of mint rewards by giving power to the minter to determine where to send the reward.

## Drawbacks

* Mint pools have existential governance issues, similar to mining pools, in which much of the blockchain governance is transfered to the pool rather than the coin holder.  While this proposal is expected to increase minter participation, it is questionable about whether that participation is truly 'distributed'.

## Dependencies

* [RFC-0002](../0002-split-coinstake-transaction/0002-split-coinstake-transaction.md)
* [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md)

## Alternatives

* [Cold Storage Minting Proposals](https://talk.peercoin.net/t/cold-storage-minting-proposal/2336), one of the highlights of which is Sigmike's cold minting proposal in which there would be a special kind of multisignature address with two private keys, one for transacting and one for minting. It is worth noting that these past proposals are compatible with the current proposed changes.
* The convention could be chosen that the mint reward must be sent to the minting address when multisignature minting. However, by giving the minter the freedom to choose the destination of the mint reward, we allow for socialization of rewards in a mint pool setting.
* Cold minting and multisig minting can be combined to provide unparalleled security and utility.  To illustrate, I will use the concept of mint keys and an address with 1 move key and 2-of-3 mint keys.  The 1 move key is cold storage and kept off-line at all times. The hot wallet will hold 2 of the mint keys and will periodically sign coinstake transactions once every 30 days (such that even the hot wallet can be airgapped). The final mint key is given to the pool operator, who has the authority to take the mint reward and socialize it with others in the pool. Combining these two methods and developing the user interface around them may drastically alter the way the average user mints, without affecting the current single-signature method.
