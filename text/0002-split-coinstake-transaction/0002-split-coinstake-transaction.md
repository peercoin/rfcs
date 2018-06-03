# Coinstake Transaction Split

- Status: proposed
- Type: enhancement
- Related components: `protocol`
- Start Date: 22-March-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Author: hrobeers

## Summary
Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction tunes the Peercoin protocol to more closely align with Bitcoin's format.
By putting the block reward back into the coinbase transaction, the coinstake transaction can be signed in advance while the coinbase transaction is signed at the time of block creation.

This RFC depends on [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md) and enables [RFC-0003](../0003-multisig-minting/0003-multisig-minting.md)

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- The verb "TO MINT" or "MINTING" refers to the action of trying to create a Block using Proof-of-Stake. "STAKING" is chosen over "MINTING" to avoid confusion with "MINING".

## Motivation
The Peercoin community has sought for a solution to *cold minting* that does not severely complicate the process of adopting code from the Bitcoin ecosystem.
While it is not entirely clear what features constitute 'cold minting', this proposal outlines a particularly protocol-sensitive path to achieve *air gapped minting*, whereby the key used at the time of minting is not sufficient to also move the coins.

Although the splitting of the coinstake transaction has been described extensively by mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81), it has never been brought up as a possible alternative for *cold minting*.
While the entire mechanisms relies on multiple RFCs, this RFC focuses on the discussion about splitting the coinstake transaction, as it is the most controversial part of the proposal.

As a historical note, discussion of this RFC shed light on the powers of the minter to create *free outputs* in the coinbase transaction, which can be fixed independently of this proposal by limiting the powers of the coinbase transaction.

## Detailed Design
Proof-of-Stake blocks in the original peercoin protocol carry the coinbase transaction as deadweight.
Only the coinbase `vin[0]` has been used to carry meta-data (e.g. signaling P2SH support). The diagram below illustrates the original block layout.

![original block layout](original.png)

Splitting the coinstake transaction into a *monetary creation* and a *coin-age consumption* transaction, has the side effect of creating small value outputs.
Because those outputs are very unlikely to mint new blocks, node implementations are adviced to join them with a future *coin-age consumption* transaction.

The minter is currently awarded a budget of 1 KB for the coinstake transaction, and will continue to have this power.  If a *coin-age consumption* transaction is greater than 1 KB, it will have to include the standard fee less 1 KB.

The diagram below illustrates the block layout with a splitted coinstake transaction.

![splitted block layout](split.png)

## Advantages

* Required for multi-signature minting, as described in [RFC-0003](../0003-multisig-minting/0003-multisig-minting.md)
* Tools that watch the coinbase transaction on Proof of Work chains like Bitcoin will be easier to adapt to Peercoin.  With this reference, this RFC should be taken as a step toward simplification, which can foster innovation.

## Drawbacks

* Dependency on [RFC-0004](../0004-remove-transaction-timestamp/0004-remove-transaction-timestamp.md).
* Hard fork of protocol rules.

## Alternatives

The *coin-age consumption* transaction could be forced to pay the standard transaction fee, which could be recovered in the *monetary creation* transaction.
As mquandalle [[1]](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81) describes, the *coin-age consumption* transaction could then be included by blocks on a competing fork, effectively destroying it's coin-age without providing a block reward.
This mechanism would penalize minters building on all forks, and would incentive to mint more actively as orphan blocks risk losing their coin-age.


## References
[1] mquandalle, Split the Peercoin coinstake transaction: [https://gist.github.com/mquandalle/7fe702a595f07f4b0f81](https://gist.github.com/mquandalle/7fe702a595f07f4b0f81)
