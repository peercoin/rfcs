# Coinstake Transaction Split

- Status: proposed
- Type: enhancement
- Related components: `protocol`
- Start Date: 22-March-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Author: hrobeers

## Summary
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction allows multiple improvements on peercoin's protocol.
Apart from aligning the coin creation with Proof-of-Work blocks, it does enable to following changes:

* Penalization of minting multiple chains. As described by mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81)
* Multi-signature minting, as described in [RFC-0003](../0003-multisig-minting/0003-multisig-minting.md)
* Alignment of the transaction format with bitcoin by moving the coinstake timestamp to the coinbase input (RFC to be created).

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- The verb "TO MINT" or "MINTING" refers to the action of trying to create a Block using Proof-of-Stake. "STAKING" is chosen over "MINTING" to avoid confusion with "MINING".

## Motivation
Multiple community members have been asking about *cold minting* and penalizing *minting multiple chains*.
While all this time, a potential solution for both those problems has been sitting in every Proof-of-Stake block, the unused coinbase transaction.
Although the splitting of the coinstake transaction has been described extensively by mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81), it has never been brought up as a possible alternative for *cold minting*. This RFC focuses on the discussion about splitting the coinstake transaction, the discussion about *multi-signature minting* is part of [RFC-0003](../0003-multisig-minting/0003-multisig-minting.md).

As mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81) describes, the *coin-age consumption* transaction can be included by blocks on a competing fork, effectively destroying it's coin-age without providing a block reward.
This mechanism penalizes minters building on all forks.

## Detailed Design
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction, has the side effect of creating a large amount of small value outputs.
Because those outputs are very unlikely to mint a new blocks, it should be allowed to join them with a future *coin-age consumption* transaction.
Because you can only find stake for a single UTXO, the protocol should specify that the *monetary creation* transaction only applies to the first input of the *coin-age consumption* transaction.
This means that a small loss in compound interest is made on the previous *monetary creation* output, but the loss can be considered negligible compared to the total minting revenue, assuming the holder actively participates in securing the blockchain.

![splitted transaction layout](split.png)

## Advantages

* Penalization of minting multiple chains (see mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81))
* Incentive to mint more actively as orphan blocks risk losing their coin-age.
* Paving the road for other improvements like multi-signature minting.

## Drawbacks

* Slight loss of compound interest as the coinbase can only be joined in the next coinstake transaction from the same script (address in case of P2PK).
* Hard fork

## Alternatives

If the community decides the penalization of orphan blocks is not wanted, the *coin-age consumption* transaction can be allowed to pay no transaction fee.
This would make it impossible for other block creators to include other minter's *coin-age consumption* transactions in their block.


## References
[1] mquandalle, Split the Peercoin coinstake transaction: [https://gist.github.com/mquandalle/7fe702a595f07f4b0f81](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81)