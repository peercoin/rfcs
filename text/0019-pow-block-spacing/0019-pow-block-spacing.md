# Proof-of-Work Block Spacing

- Status: proposed
- Type: protocol adjustment
- Related components: 
- Start Date: 05-04-2020
- Discussion: 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by:
- Author: Nagalim

## Summary

The current PoW block spacing is a complicated function involving the frequency of PoS block generation.
This can have unintended consequences on PoW block generation, and is overall more complicated than needed.
This proposal will fix the PoW block target to 60 minutes, which is the approximate average of current PoW block spacings.
This will greatly simplify a number of concerns regarding PoW difficulty and block frequency.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

We have seen a number of instances in recent times of rapid PoW block creation that does not increase the PoW difficulty the way it intuitively should.
Ideally, when PoW blocks are formed in rapid succession, the PoW difficulty will increase to compensate and slow down the rate of block formation.
With the current complexity of PoW block target, this is not always the case, and sometimes the difficulty will even go down when PoW blocks are found one right after another.
A simplification of the PoW block target to a fixed 60 minutes will mend this situation, and will always result in an increase of PoW difficulty when blocks are found more frequently than the 60 minute spacing.

As a side benefit, this modification will make it easier to compute expected miner rewards, as well as average total block spacing.
This will ultimately make it easier to use the chain by various third parties, which may grow the utility of the chain for those that mine and transact.

In addition, it is counter-intuitive to increase coin distribution (the main function of PoW block generation) when PoS block generation slows down.
A changing PoW block target makes it hard to estimate a total coin inflation and does not seem in line with the core philosophies of the Peercoin chain.

## Detailed design

The PoW block target will be fixed to 60 minutes, despite PoS block spacing.

## Drawbacks

The reason the current protocol was chosen was so that PoW would replace PoS during periods where PoS drops out of the network for whatever reason.
While on its face this may seem like a good thing, ultimately PoW blocks are not secure and can easily be overwritten by the first PoS block to be added to the network due to the fact that PoW blocks have minimal chain weight.
The chain cannot be considered secure during periods where PoS is not functioning properly, and it is not good practice to increase the frequency of PoW block generation during these times, because there is little protection against a double spend.

## Alternatives

Network events where PoS drops out are discontinuous, meaning that either there is an issue affecting a vast number of nodes, or there is not.
An alternative implementation that would maintain the spirit of having PoW step in when PoS drops out is to make a discontinuous PoW target that is 60 minutes most of the time, but changes if there has been a long time with no PoS blocks.
For example, reducing PoW block target to 20 minutes if there has not been a PoS block in 150 minutes would achieve such a paradigm.

## Unresolved questions?

