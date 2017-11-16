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
Multiple community members have been asking about *cold minting* and the power of minters to add *free transactions* to their blocks.
While all this time, a potential solution for both those problems has been sitting in every Proof-of-Stake block, the unused coinbase transaction.
Although the splitting of the coinstake transaction has been described extensively by mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81), it has never been brought up as a possible alternative for *cold minting*. This RFC focuses on the discussion about splitting the coinstake transaction, the discussion about *multi-signature minting* is part of [RFC-0003](../0003-multisig-minting/0003-multisig-minting.md).

Minters are able to include transaction data without paying the fee when they create a block, limited only by the blocksize.
As a minter of a small number of blocks might not hold many coin at stake, this can be a method by which a bad actor can drastically bloat the chain without much personal loss.
When splitting the coinstake transaction, a natural limit of 1 free KB per minted block will provide minters with flexibility to adjust their outputs as desired while preventing spam.

## Detailed Design
Proof-of-Stake blocks in the original peercoin protocol carry the coinbase transaction as deadweight.
Only the coinbase `vin[0]` has been used to carry meta-data (e.g. signaling P2SH support). The diagram below illustrates the original block layout.

![original block layout](original.png)

Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction, has the side effect of creating small value outputs.
Because those outputs are very unlikely to mint new blocks, node implementations are adviced to join them with a future *coin-age consumption* transaction.

The minter is awarded a budget of 1 KB for the *coin-age consumption* transaction in order to adjust their UTXO table as they see fit.
If the *coin-age consumption* transaction is greater than 1 KB, it will have to include the standard fee less 1 KB.

The diagram below illustrates the block layout with a splitted coinstake transaction.

![splitted block layout](split.png)

## Advantages

* Paving the road for other improvements like multi-signature minting.
* Limiting the power of minters to spam the blockchain.

## Drawbacks

* Hard fork

## Alternatives

The *coin-age consumption* transaction could be forced to pay the standard transaction fee, which could be recovered in the *monetary creation* transaction.
As mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81) describes, the *coin-age consumption* transaction could then be included by blocks on a competing fork, effectively destroying it's coin-age without providing a block reward.
This mechanism would penalize minters building on all forks, and would incentive to mint more actively as orphan blocks risk losing their coin-age.


## References
[1] mquandalle, Split the Peercoin coinstake transaction: [https://gist.github.com/mquandalle/7fe702a595f07f4b0f81](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81)
