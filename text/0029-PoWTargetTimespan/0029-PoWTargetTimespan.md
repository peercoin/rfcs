# Proof-of-Work Difficulty on PoS Blocks

- Status: proposed
- Type: protocol adjustment
- Related components: pow.cpp, chainparams.cpp
- Start Date: 01-28-2024
- Discussion: Discord
- Supersedes: 
- Superseded by:
- Author: Nagalim

## Summary

Peercoin's unique approach includes Proof of Stake and Proof of Work elements, the former to secure the chain and the latter to perform fair initial distribution.
The difficulty adjustment algorithm (DAA) is used to target a block time for each of these elements.
Blockchains like bitcoin use a 'Median Time Past' DAA every 2016 blocks.
Peercoin uses an exponential moving average (EMA) where the smoothing `nInterval` in pow.cpp is calculated from the `nTargetTimespan` in chainparams.cpp.
This proposal seeks to separate the PoS and PoW target timespans, keeping the PoS timespan the same but making the pow timespan quicker.
As such, we will introduce `nPoWTargetTimespan`, and calculate a different `nInterval` depending on whether `fProofOfStake` is true or not.
This is not to be confused with Bitcoin's `nPoWTargetTimespan`, which is the 2016 block period of a 'Median Time Past' DAA and has no bearing on this RFC except as context.
We will make an argument in this RFC for a 36-hour `nPowTargetTimespan` as an ideal number to keep second-order effects trivial.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

While Peercoin's PoS has an initial maturation period on the order of `4*nTargetTimespan`, Peercon's PoW has no such sunk cost of time.
As such, PoW hashpower comes and goes freely, often oscillating dramatically within the period of a day.
Using a faster EMA will allow the protocol to adjust more easily to such changes.
The main fear, that of overzealously turning up the difficulty and causing an oscillating difficulty bomb, is mostly diffused due to PoW difficulty changes occuring on PoS blocks through RFC-0020.
As such, aside from implementation, we will mainly concern ourselves with the intricacies of how a faster `nPoWTargetTimespan` interplays with changing PoW difficulty on PoS blocks.

## Detailed design

(At time of writing) Line 74 of chainparams.cpp:

`        consensus.nTargetTimespan = 7 * 24 * 60 * 60;  // one week`

would be followed by:

`        consensus.nPoWTargetTimespan = 36 * 60 * 60;  // one and a half days`

(At time of writing) Line 53 of pow.cpp:

`        int64_t nInterval = params.nTargetTimespan / nTargetSpacing;`

could be followed by:

`    if (!fProofOfStake && IsProtocolV14(pindexPrev)`

`        int64_t nInterval = params.nPoWTargetTimespan / nTargetSpacing;`

This is not the only possible implementation method.

## Drawbacks

### Drawback 1: Basic Concerns
Slight protocol complexity increase.  Also, the coincidental naming of `nPowTargetTimespan` with Bitcoin's protocol may cause confusion, but is the name that follows convention.

### Drawback 2: RFC-0020 Cross Term
PoW difficulty change on PoS blocks (rfc-0020) introduces a cross term in the EMA that is 'second order'.
This is due to splitting long PoW difficulty changes into two components: before the PoS block preceding the discovered PoW block, and the short span after.
This can be compared to 'before rfc-0020' like this:

`[(t-1)+2(a+b)]/(t+1) = [(t-1)+2a]/(t+1)*[(t-1)+2b]/(t+1) - 2ab/(t+1)^2`

The `2ab` cross term is at second order in `1/(nInterval+1)`.
We often assume `a` to be larger than `b`, as `a` represents a distance on order of PoW block spacing and b represents a distance on order of PoS block spacing, but in full generality we cannot assume such things.
Still, this gives us an order of magnitude to examine, and pursue a time interval t (in units of hours because that is the PoW target spacing) such that:

`2 * 1 hour * 0.1667 hours / (t hours + 1)^2 << 1`

With `t=36 hours`, this comes out to `2.435e-4 << 1`, which is appropriate.
We can even imagine an extreme case where PoS and PoW blocks are both an order of magnitude off target at the same time, and still we would be at `0.02435 << 1`.

### Drawback 3: Nonlinearity
The bare minimum of nInterval is 1 block, wherein we would straightforwardly multiply the difficulty by the distance of the actual time away from the target time.
Here we can see an issue, let us imagine a two hour period with 2 possibilities.
In one case, the blocks are found at 1 hour intervals and the difficulty is unchanged.
In the other case, the blocks are found at 0.5 and 1.5 hour intervals.
In calculating the resulting difficulty in this 1-block nInterval case, we get:

`0.5 * 1.5 = 0.75 != 1`

As such, we see that having an nInterval that greatly exceeds the block spacing is necessary to avoid additional nonlinear effects.

`t hours >> 1`

36 Seems like a safer number here than 24.

## Alternatives

Let us perform the above calculations for 24 hours.
The equation is a second order equation, so we should expect `1.5^2=2.25` increase in cross term.

`2 * 1 hour * 0.1667 hours / (24 hours + 1)^2 = 5.334e-4`.

As such, 24 hours also appears to be a viable alternative that will not make the cross term significant.
However, this does not negate the additional nonlinearity effects.

## Unresolved questions?

How much nonlinearity can we tolerate?
We are trying to ensure that rapidly oscillating hash power does not substantially alter the average block time.
The reason this is important in Peercoin is because while Bitcoin uses `Median Time Past` DAA, Peercoin uses an EMA.
As such, the overall median time is not assured, given by the above discussions on nonlinearity when nActualSpacing is on the order of nPoWTargetTimespan.
The author has posited that 36 >> 1, choosing a human-relevant number corresponding to 1.5 days.
This is clearly not exact, but is there any definitive calculation that can be done to accurately define how much error in median block time we can tolerate?
