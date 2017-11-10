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

* Multi-signature minting, as described in [RFC-0003](../0003-multisig-minting/0003-multisig-minting.md)
* Alignment of the transaction format with bitcoin by moving the coinstake timestamp to the coinbase input (RFC to be created).
* Provides easily manageable limitations on the power of minters to make free transactions.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- The verb "TO MINT" or "MINTING" refers to the action of trying to create a Block using Proof-of-Stake. "STAKING" is chosen over "MINTING" to avoid confusion with "MINING".

## Motivation
Multiple community members have been asking about *cold minting* and *gamifying UTXO optimization*.
While all this time, a potential solution for both those problems has been sitting in every Proof-of-Stake block, the unused coinbase transaction.
Although the splitting of the coinstake transaction has been described extensively by mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81), it has never been brought up as a possible alternative for *cold minting*. This RFC focuses on the discussion about splitting the coinstake transaction, the discussion about *multi-signature minting* is part of [RFC-0003](../0003-multisig-minting/0003-multisig-minting.md).

Minters are able to include transaction data without paying the fee when they create a block, limited only by the blocksize.
By dividing their stake into many small UTXOs for free in a created block, minters can gamify and optimize their block generation rate, giving them an advantage in generating blocks more rapidly.
When splitting the coinstake transaction, a natural limit of 1 free KB per minted block will provide minters with flexibility to adjust their outputs as desired while preventing spam and gamification of UTXO optimization.

## Detailed Design
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction, has the side effect of creating a large amount of small value outputs.
Because those outputs are very unlikely to mint a new blocks, it should be allowed to join them with a future *coin-age consumption* transaction.
Because you can only find stake for a single UTXO, the protocol should specify that the *monetary creation* transaction only applies to the first input of the *coin-age consumption* transaction.
This means that a small loss in compound interest is made on the previous *monetary creation* output, but the loss can be considered negligible compared to the total minting revenue, assuming the holder actively participates in securing the blockchain.

![splitted transaction layout](split.png)

The minter is awarded a budget of 1 KB for the *coin-age consumption* transaction in order to adjust their UTXO table as they see fit.
If the *coin-age consumption* transaction is greater than 1 KB, it will have to include the standard fee less 1 KB.

## Advantages

* Incentive to mint more actively as orphan blocks risk losing their coin-age.
* Paving the road for other improvements like multi-signature minting.
* Limiting the power of minters to gamify the UTXO table.

## Drawbacks

* Slight loss of compound interest as the coinbase can only be joined in the next coinstake transaction from the same script (address in case of P2PK).
* Hard fork

## Alternatives

The *coin-age consumption* transaction could be forced to pay the standard transaction fee.
As mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81) describes, the *coin-age consumption* transaction could then be included by blocks on a competing fork, effectively destroying it's coin-age without providing a block reward.
This mechanism would penalize minters building on all forks.


## References
[1] mquandalle, Split the Peercoin coinstake transaction: [https://gist.github.com/mquandalle/7fe702a595f07f4b0f81](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81)
