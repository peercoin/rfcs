# Remove Proof-of-Work Block Signature

- Status: implemented
- Type: enhancement
- Related components: `protocol`
- Start Date: 21-November-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: hrobeers

## Summary

Peercoin blocks contain a block signature covering it's contents, as is required by the Proof-of-Stake algorithm.
Unfortunately this block signature requirement has also been imposed on Proof-of-Work blocks, while these don't strictly need the signature as the Proof-of-Work covers the entire block's content.
This RFC proposes the removal of the requirement for a valid signature in Proof-of-Work blocks.


## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


## Motivation

The block signature requirement imposes that at least one hot key should be loaded into the node's wallet in order to be able to produce valid blocks.
For Proof-of-Stake blocks this is a fair assumption as a key is needed to prove ownership over stake, but for Proof-of-Work blocks this requirement adds unneeded overhead and complicates mining to offline keys.

Proof-of-Work mining nodes cannot disable the wallet functionality in their node like is possible on Bitcoin, and need to make sure the wallet is unlocked and contains the private key required to sign a block once found.
Since pool nodes are an attractive target of attack due to the large turnover they can generate, some pools prefer to mine to cold key or at least keys not present in the mining node's wallet.

Therefore some pools deploy a clever trick to mine directly to offline keys.
Because the block signature is required to match the key for vout[0], [Peercoin Solo Pool](https://hrobeers.github.io/ppc-solo-mine/) mines a zero value output to vout[0] and sends the full reward to offline keys via vout[1] and vout[2].
Block `0000000000000001b4ec9c4661aab9856228e74f16e842b42a4952527d7697fe` on Peercoin's mainnet illustrates this trick, and serves as an extra argument for [RFC-0005: Unspendable Zero Outputs](../0005-unspendable-zero-outputs/0005-unspendable-zero-outputs.md).

By removing the valid block signature requirement for Proof-of-Work blocks, Proof-of-Work mining pool operation will be simplified and more secure without the need to deploy clever tricks to circumvent hot storage of mined funds.


## Detailed design

- Remove the singing step from the BitcoinMiner thread for Proof-of-Work blocks.
- Remove the signing step in the submitblock rpc call for Proof-of-Work blocks.
- Remove the validation step of the block signature for Proof-of-Work blocks.

A hard-fork is required since older nodes will invalidate Proof-of-Work blocks without a valid block signature.

## Drawbacks

Why should we *not* do this?

- hard-fork

## Alternatives

What other designs have been considered? What is the impact of not doing this?

- [RFC-0005](../0005-unspendable-zero-outputs/0005-unspendable-zero-outputs.md) minimises the impact of the zero value output trick on the memory consumption of network nodes. However the author believes this RFC is still relevant as it simplifies the Proof-of-Work mining operation.

## Unresolved questions

What parts of the design are still to be done?
