# PoWTargetTimespan

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
The PoW DAA will be moved to what is referred to as "absolutely scheduled exponentially rising targets" (ASERT) [https://toom.im/files/da-asert.pdf], as implemented by Bitcoin Cash (BCH).
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
ASERT allows for a more stable DAA when responsivity is turned up, due to the basic property that `2^(a)+2^(b)=2^(a+b)` without the nonlinearities described in RFC-0029.
The non-recursive nature of ASERT will also make RFC-0020, PoW difficulty on PoS blocks, easier to implement by allowing for intermediate difficulty changes without iterating.
BCH has done quite a bit of research on this topic, having implemented aserti3-2d in November 2020 in the Bitcoin Cash ABC fork.
However, the context of 1 hour target blocks and PoS intermediate difficulty adjustments makes Peercoin's control system unique from BCH.
Therefore, careful consideration should be paid to selecting the half-life 'tau'.

## Detailed design

Copy from BCH, then add in RFC20.
12 Hour 'tau' will be used in the main text.

## Drawbacks

### Drawback 1: Basic Concerns

Significant increased protocol complexity, but has been implemented already on a Bitcoin fork.
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

Choice of 'tau': 6, 12 hours, 24 hours, 48 hours.

## Unresolved questions?

