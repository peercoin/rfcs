# Proof-of-Work Difficulty on PoS Blocks

- Status: implemented
- Type: protocol adjustment
- Related components: 
- Start Date: 07-17-2020
- Discussion: 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by:
- Author: Nagalim

## Summary

Peercoin uses PoW block rewards as a fair distribution method for new coins.
However, the chain is fundamentally secured by PoS blocks, not PoW.
As such, a large chain of PoW blocks is detrimental to the security of the chain.
So called 'difficulty bombs' often result in long periods of time with no PoW blocks, followed by an abrupt decrease in PoW difficulty and then several PoW blocks found in a row, which momentarily compromises chain security.
By adjusting the PoW difficulty on PoS blocks found during these long periods, the difficulty bomb is diffused and PoW blocks can be found on more regular intervals.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Peercoin shares a mining algorithm with Bitcoin, so it is assumed that the total potential hashpower in existance greatly exceeds the hashpower targeting the Peercoin chain.
As a result, miners will turn to Peercoin when profit is high, drive up the PoW difficulty, then move on to other chains.
This causes a difficulty bomb in which the difficult remains high until the next PoW block is found.
Once that next block is found at the high difficulty, the difficulty immediately drops and the next several PoW blocks are easily found until the difficulty is driven up again.
This boom-and-bust behavior causes long strings of PoW blocks that have low chainweight and can be orphaned by a single PoS block.
This behavior is exacerbated by longer block spacing targets, and has seen a resurgance since the inclusion of RFC-0019.

## Detailed design

Difficulty adjustments are calculated via the following algorithm:

New Mining Difficulty = Previous Mining Difficulty * Lambda

Where Lambda is the target time divided by the time it took to mine the previous block.
Lambda is a number near 1 that is bounded directly in the protocol.
We will not make any consequential changes during the interval where Lambda is less than 1 (i.e. when it has been less than the target time since a PoW block was found).

If a PoS block is found before the target time since the last PoW block, the PoW difficulty will be the same as the last PoW block (i.e. Lambda has a minimum of 1 on PoS blocks).
Each time a PoS block is found after the target time since the last PoW block, a new mining difficulty will be used with Lambda calculated using the time of the last PoW block and bounded appropriately.
In order for a new PoW block to be found, it must only beat the difficulty implied by the last PoS block, though the new PoW difficulty will still be calculated using the last PoW block.

## Drawbacks

Allowing difficulty to drop during PoS blocks should be effective at diffusing difficulty bombs, up to the bounds on Lambda.
Still, the natural boom-and-bust caused by convenience of miners to redirect their hashpower in bursts cannot be expected to be solved on-chain as long as Peercoin shares a hashing algorithm with more hash-dominant coins such as Bitcoin, and switching to a unique hashing algorithm is contrary to the energy-efficient design of Peercoin.
While this RFC is not a perfect answer for the boom-and-bust behavior, this change is expected to be an improvement.

The increased protocol complexity and upkeep of its functions is the only drawback to this proposal.

## Alternatives

## Unresolved questions?

