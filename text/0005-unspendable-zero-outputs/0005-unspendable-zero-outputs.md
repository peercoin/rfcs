# Unspendable Zero Outputs

- Status: proposed
- Type: bugfix (memory leak)
- Related components: `protocol`
- Start Date: 20-November-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: hrobeers

## Summary

Peercoin requires outputs to have a value of at least 0.01 PPC.
It's unclear to the author why this value has been chosen, but it seems a reasonable dust prevention measure.
However, there is one exception to this rule, 0 PPC outputs are allowed.
Such zero-value outputs can have some useful applications laid out below, but an unfortunate side-effect is that they take up memory in the node's UTXO table forever, increasing it's memory usage.
This RFC proposes a solution that fixes this memory leak without changing the rules for output values.


## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


## Motivation

Zero value outputs have many interesting applications on Peercoin, such as 'data carrying outputs (OP_RETURN)', 'free transaction tagging ([P2TH](https://peerassets.github.io/P2TH/))`, 'Peercoin PoW mining to cold storage', ... .
Data carrying ouputs (OP_RETURN) are unspendable and therefore can safely be pruned from the UTXO table, which allows nodes to free memory that won't be needed anyway.
But unlike those data outputs, other zero value outputs are spendable, and therefore nodes cannot safely prune those outputs as they would risk forking of when someone spends such an output which they've marked as spent by pruning them.

Transaction `6f224c92c32cc2bd1146b677acca0d823b6c1f258eef820732081b493d51fa13` on the peercoin testnet shows that it is possible to spend a zero output.
Any node that would have pruned this output from it's UTXO table would have invalidated this transaction resulting in a fork between zero output pruning and non pruning nodes.


## Detailed design

The implementation of a simple soft-fork of the protocol rule to make zero value outputs unspendable is trivial.

If needed this change can be soft-forked without much effect on nodes running older versions of the protocol.
Old nodes will still produce valid blocks as long as no transaction is included that spends a zero value output.
Transaction spending zero value outputs are very rare as it increases the transaction size, and therefore it's fee, while not adding any monetairy value to the transaction.

## Drawbacks

Why should we *not* do this?

## Alternatives

What other designs have been considered? What is the impact of not doing this?

- Moving to a different way of validating transactions that does not use an UTXO table (e.g. [BITCRUST](https://bitcrust.org/)), could circumvent this memory leak. However, the author believes the protocol should take a sensible position on how to handle zero value outputs rather than pushing node software towards a specific implementation.

- By not implementing this change users can spam the UTXO table cheaply with zero value outputs, where no incentive exists to clean them up (spend them) as they contain no value. Luckily this solution, irrespective of it's activation time, frees all memory occupied by zero value outputs starting from the genesis block.

## Unresolved questions

What parts of the design are still to be done?
