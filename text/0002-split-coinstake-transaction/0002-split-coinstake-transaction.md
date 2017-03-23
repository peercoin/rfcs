# Split Coinstake Transaction & Multisig Minting

- Status: proposed
- Type: enhancement
- Related components: `protocol`
- Start Date: 22-March-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Author: hrobeers

## Summary
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction has multpile uses and advantages.

* It enables *Cold Minting* by allowing multi-signature *coin-age consumption* transactions to be partially pre-signed off-line.
* Depending on the implementation it can allow the penalization of minting multiple chains. As described by ...

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- The verb "TO MINT" or "MINTING" refers to the action of trying to create a Block using Proof-of-Stake. "STAKING" is chosen over "MINTING" to avoid confusion with "MINING".

## Motivation
Multiple community members have been asking about *cold minting* and penalizing *minting multiple chains*.
While all this time, a potential solution for both those problems has been sitting in every Proof-of-Stake block, the unused coinbase transaction.

Although the splitting of the coinstake transaction has been described extensively by mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81), it has never been brought up as a possible alternative for *cold minting*.
Therefore, this article mainly covers the *cold minting* alternative, but the *minting multiple chains* discussion should not be ignored.

Multi-signature minting requires minimal changes to the protocol and allows coins secured by off-line or hardware wallets to actively participate in the minting process.
A 2-of-3 multisignature script can be composed using one mint key and two off-line keys, so in order to spend the coins one at least needs access to one of the off-line keys.
The multi-signature minting process would require the holder to pre-sign the *coin-age consumption* transaction off-line and send it off to his minter to find stake for that output using the minting key.
The on-line minter would be unable to spend the coin in any other way that broadcasting the *coin-age consumption* transaction, that returns all funds to sender only consuming the coin-age and a transaction fee.

## Detailed Design

### Splitting the coinstake
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction requires only the *coin-age consumption* transaction to be fully signed by the minter.
Because the block reward amount and the creation timestamp are moved to the *monetary creation* transaction, taking the place of the currently unused coinbase transaction, these properties are excluded from the *coin-age consumption* signature.
Therefore, the minter can modify the coinbase timestamp and block reward without having to update the *coin-age consumption* signature, allowing him to find stake for an externally signed *coin-age consumption* transaction.

To reduce the loss of compound interest due to the creation of a small coinbase output,

### Multi-signature minting
Multi-signature minting is not possible in the current protocol because the block signature is required to match the public key of the coinstake transaction.
Adding a simple rule for multi-signature scripts that blocks should be signed by one of it's public keys would enable multi-signature minting.

### Example scenarios
TODO

## Advantages

* More secure minting.
* Permanent outsourcing of minting is not possible, every mint needs to be pre-signed by the holder.
* Minimal protocol changes.
* Penalization of minting multiple chains (see mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81))
* Incentive to mint more actively as orphan blocks risk losing their coin-age.

## Drawbacks

* Slight loss of compound interest as the coinbase can only be joined in the next coinstake transaction from the same script (address in case of P2PK).
* Hard fork

## Alternatives

* One could allow the pre-signed coinstake transaction to not pay the fee that was compensated in the coinbase, making it impossible to consume it's coinage outside of the coinstake transaction.
This effectively results in disabling the proposal of mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81).


## References
[1] mquandalle, Split the Peercoin coinstake transaction: [https://gist.github.com/mquandalle/7fe702a595f07f4b0f81](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81)