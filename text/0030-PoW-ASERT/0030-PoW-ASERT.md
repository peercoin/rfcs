# PoW ASERT

- Status: proposed
- Type: protocol adjustment
- Related components: pow.cpp, chainparams.cpp
- Start Date: 02-04-2024
- Discussion: Discord
- Supersedes: RFC0029
- Superseded by:
- Author: Nagalim

## Summary

Peercoin's unique approach includes Proof of Stake (PoS) and Proof of Work (PoW) elements, the former to secure the chain and the latter to perform fair initial distribution.
The difficulty adjustment algorithm (DAA) is used to target a block time for each of these elements.
Blockchains like bitcoin use a 'Median Time Past' DAA every 2016 blocks.
Peercoin uses an exponential moving average (EMA) where the smoothing `nInterval` in pow.cpp is calculated from the `nTargetTimespan` in chainparams.cpp.
This proposal will separate the PoW and PoS DAA, keeping the PoS the same.
The PoW DAA will be moved to what is referred to as "absolutely scheduled exponentially rising targets" [ASERT](https://toom.im/files/da-asert.pdf), as implemented by Bitcoin Cash (BCH).
Block spacing, block reward, and resulting inflation should all remain theoretically the same on average across this change, as the target hashpower-to-difficulty ratio should remain the same.
The main change is how the target is tracked.

The key parameter is the smoothing timescale "tau", which is the half-life of the DAA.
This means that the PoW difficulty will be cut in half after the time 'tau' has passed with no PoW block.
It also means that 'tau' number of blocks will double the difficulty (in units of target spacing or, at time of writing, in hours) if found in immediate succession.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

While Peercoin's PoS has an initial maturation period on the order of `4*nTargetTimespan`, Peercon's PoW has no such sunk cost of time.
As such, PoW hashpower comes and goes freely, often oscillating dramatically within the period of a day.
Using a more responsive DAA will allow the protocol to adjust more easily to such changes.
ASERT allows for a more stable DAA when responsivity is turned up, due to the basic property that `2^(a)*2^(b)=2^(a+b)` without the nonlinearities described in RFC-0029.
The non-recursive nature of ASERT will also make RFC-0020, PoW difficulty on PoS blocks, easier to implement by allowing for intermediate difficulty changes without iterating.
BCH has done quite a bit of research on this topic, having implemented aserti3-2d in November 2020 in the Bitcoin Cash ABC fork.
However, the context of 1 hour target blocks and PoS intermediate difficulty adjustments makes Peercoin's control system unique from BCH.
Therefore, careful consideration should be paid to selecting the half-life 'tau'.

## Detailed design

We can mostly just look to the [BCH pow.cpp](https://github.com/bitcoin-cash-node/bitcoin-cash-node/blob/master/src/pow.cpp) code, then add in the variables needed to chainparams.cpp.
When doing this, other than merging Peercoin's PoS and older forms of PoW for chain validity, there are a few additional considerations that need to be dealt with.

Firstly, rfc-0020 can quite easily be added to ASERT by taking the [time_delta](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/2020-11-15-asert.md) from any block, PoS or PoW, rather than asking for explicitly one or the other.
In the current Peercoin code, this comes down to the line:
`const CBlockIndex* pindexPrev = GetLastBlockIndex(pindexLast, fProofOfStake);`
Where we would not check for fProofOfStake, but instead take either PoS or PoW for the current blocktime.

However, this brings us to the second issue which is the requirement of a new variable `nPoWHeight`.
This is essential to the ASERT algorithm and is how it follows mean-reversion, such that the difficulty of one block is not dependent on the difficulty preceding it, but rather refers back to an 'anchor' block.
The anchor block is chosen at the fork, and nPoWHeight must be counted either from it, or computed from the start of the chain and the nPoWHeight of the anchor block subtracted from it.
If nPoWHeight is counted from the fork, then no re-scan of the chain should be necessary on upgrade, but it is a less appealing number as a chain variable.

The entirety of the ASERT code should be diligently inspected for where the PoS vs PoW pindex's need to be called.
It is nontrivial to inspect, but should not require much additional code or effort to point to the correct one at the correct time.
For example, the choice of an anchor block will require reference to 'fProofOfStake' in order to pick a PoW block and not a PoS one.

## Drawbacks

### Drawback 1: Basic Concerns

Significantly increased protocol complexity, but has been implemented already on a Bitcoin fork.
Code for both EMA and ASERT will need to be retained, both for the PoS/PoW separate DAAs, but also for validation of past blocks.

### Drawback 2: Unfamiliarity

The ASERT protocol is newer than Peercoin's EMA, and could perform differently than expected.
When selecting protocol parameters, theoretical modeling and intuition can both only go so far in predicting the future.
However, as long as the chain is not fundamentally destabilized, the parameter 'tau' can be tuned fairly easily in future hard forks if necessary.

### Drawback 3: Nonlinearity

Rather than the algorithmic nonlinearities discussed in RFC-0029, here the nonlinearities are in the behavior of the control loop.
All control loops are succeptible to differential nonlinearities (DNL) and integral nonlinearities (INL).
The DNL noise floor tends to be the dominant factor, and can be determined using the standard deviation of an exponential block interval distribution (which we assume to be 1 hour or 3600 seconds).
Taking 'tau' to be 12 block lengths, we get a DNL noise floor of `1*sqrt(1/12) = 0.2887 hours`, or 1040 seconds.
This is ~29% noise compared to the block spacing, which is of course significantly higher than in BCH where they have ~6% with a 35.3 second noise floor with 10 minute blocks and a 'tau' of 288 block lengths.
INL will be more responsible for long term statistics such as PoS:PoW block ration and overall PoW inflation rate.
However, BCH showed that ASERT block spacing tends to be fairly insensitive to even high INL noise.

## Alternatives

In choosing the ideal value of tau, we will consider 4 options: 6, 12, 24, or 48 hours.
The implementation chosen by BCH involved a tau of 48 hours, after performing various analysis and tests of the statistical noise.
We use a similar justification as RFC-0020 in that the more common PoS blocks offset the noise of the PoW blocks.
We care less about random fluctuations on the order of the PoW block spacing, and more about ensuring we don't often get 3 or more PoW blocks within the *PoS* block spacing.
As such, the standard methods of noise analysis for a control system do not pertain here without modification.

The noise level, as described in "Drawback 3: Nonlinearity" of this RFC, can be reimagined using the PoS block spacing for the noise floor, then ask about getting e.g. a 6-block PoW string.
This gets complicated because this noise floor is itself noisy because the PoS blocks also have a DAA control loop.
But let's take just take some simplistic assumptions, calling upon the PoS block spacing twice: once for reducing the noise of the control loop through RFC-0020, and again for the minimum frequency where we care about having too many pow blocks.
E.g. we do not care so much if we get a string like: PoW-PoW-PoS-PoW-PoW-PoS-PoW-PoW, even if the timestamps in this case would normally be untentable to the standard analysis of noise levels used by the BCH community to characterize ASERT.

As such, we will use a DNL noise floor of `sqrt(0.1667/Tau)` to correspond to the 1/6 target block spacing for the PoW:PoS ratio through RFC-0020.
Next, we will seek the overlap between this shot noise and the occurance of PoS blocks at 1/root(Hz): `sqrt(0.1667)`.
This presents us with a figure of merit: `x = PoW/PoS_block_ratio * sqrt(1/Tau)`.
This figure is in some regards a kind of probability for collision within the framework of the dual nature of the blocks.
We will take e.g. 6 periods and ask for the likelihood of any collision: `1-(1-x)^6`

Some values include, when PoS blocks are 10 min, in format `{Tau hours; 6 period collision likelihood}`:
`{6;34.4%},{12;25.6%},{24;18.8%},{48;13.6%}`.
If PoS block times are 5 min: `{6;18.8%},{12;13.6%},{24;9.8%},{48;7.0%}`

On the other hand, we can just look empirically at the longer end of Tau and ask how it would respond to abrupt changes in hash rate.
`2**(1/48) - 1 = 1.45%` is the most the difficulty can increase for 2 blocks with 0 time between them for BCH's choice of 48 hour Tau.
The current EMA DAA allows for change of `(tau_ema - 1) / (tau_ema + 1) - 1 = 1.2%` for `tau_ema = 7 days`.
However, if we look empirically at the required difficulty changes due to price action on either Peercoin or Bitcoin networks (due to the shared sha-256 algorithm), we can easily see changes of 10% within a short timespan required.
While we do not want to take empirical evidence of price change as gospel, we can certainly talk about how to achieve a change on the order of 10% within a few blocks.

We seek e.g. the condition: `2**(1/Tau) - 1 = 10% T.F. Tau = 7.3`.
This implies that if we want these kinds of changes to happen after just a single pair of blocks, we need to tune all the way down to our lower limit.
However, we have tolerance for a pair of PoW blocks, as was already stated.
Because of the nature of the function, to generalize we can multiply this condition by the number of blocks we have tolerance for (minus 1).
If we then apply a sense of rounding for human readability, we can somewhat clearly state an alternative:
'If we can tolerate 3 PoW blocks in a row during dramatic price changes, then Tau = 12 hours is a good choice.
If we can allow 4 or 5 PoW blocks in a row during extreme events, then Tau = 24 hours would be a more overall stable choice.'

## Additional Notes on Protocol Choices

A choice was made in this text with regards to implementation of RFC-0020.
The way that RFC was imagined, it would only be applied to PoW blocks with time greater than the target.
Here, we have offered a slightly alternative implementation.
Upon finding a block, the difficulty of ASERT normally changes discontinuously:
`exponent = (time_delta - ideal_block_time * (height_delta + 1)) / halflife`.
On a new PoW block, both the `time_delta` and the `height_delta*ideal_block_time` change.
On a new PoS block in this implementation, the time_delta will change and lower the difficulty no matter what.
The effect of this is that PoS blocks almost always lower PoW difficulty and PoW blocks almost always raise PoW difficulty.

While this is not how RFC-0020 was imagined, it seems a very elegant and simple implementation that functions quite well and can be communicated simply:
`PoS blocks lower PoW difficulty, PoW blocks raise PoW difficulty.`
