 PoW Spacing Augmentation

- Status: proposed
- Type: protocol adjustment
- Related components: 
- Start Date: 09-23-2022
- Discussion: 
- Supercedes: (fill me in with a link to RFC this supersedes - if applicable)
- Requisite: rfc-0020
- Author: Nagalim

## Summary

The purpose of Proof of Work in the Peercoin chain is somewhat unique in its design.
Specifically, the purpose is to provably distribute coin rather than to validate an immutable record.
Given the dependency of rfc0020 regarding updating the PoW difficulty on PoS blocks, the protocol has the opportunity to tighten up difficulty changes without fearing the backlash of an inescapable difficulty bomb.
With this in mind, we revisit the difficulty adjustment algorithm and look for opportunities to achieve a more stable chain with regular distribution.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Strings of PoW blocks are bad and no one likes them.

Blockchain stability and the concept of finality rely on a regular distribution of blocks.
Long strings of PoW chains can be orphaned with just a single PoS block, damaging both the integrity of the chain and the miners who worked for the blocks.
With this in mind, we seek methods of discouraging multiple PoW blocks in a row.
At the same time, we do not want to affect the rewards and behaviors of the PoS mechanism, so we will not couple any portion of the PoW algorithm to awareness of PoS blocks, aside from the safe hallmarking of time as in rfc20.
Rather, we will use the passage of time as our sole feedback control and we will identify an appropriate dilation of the difficulty adjustment algorithm to accentuate difficulty spikes on tightly packed PoW blocks.

## Detailed design

The current algorithm for modifying the proof of work difficulty is an exponential moving average as follows:

        int64_t nInterval = params.nTargetTimespan / nTargetSpacing;
        bnNew *= ((nInterval - 1) * nTargetSpacing + nActualSpacing + nActualSpacing);
        bnNew /= ((nInterval + 1) * nTargetSpacing);

This is an exponential moving average retargeting algorithm with an nInterval targeting a 1 week timespan (normalized by the target spacing of 1 hour).
The algorithm used in pow.cpp is common between PoW and PoS difficulty adjustments, albeit with a different target spacing.
This target spacing is set separately for PoS and PoW difficulty adjustments just before the algorithm via 'if (fproofofstake)'.
The 'timespan' is chosen to be a week as a global parameter throughout the code.
Having a simple determination of the timespan and the target, and therefore 'nInterval', is important for chains that use difficulty as a form of chainweight.
The chainweight in Peercoin, however, is not a function of PoW difficulty at all.
This is a degree of freedom we can leverage to accomplish the goal of more even block distribution.

In considering concepts of tuning the difficulty adjustment algorithm, we consider the natural asymmetry of the adjustment.
Across a single block, the difficulty can move infinitely far down (in the case of eternity between blocks) but can only move up as quickly as the limit of nActualSpacing=0, which is around a 1% change per block.
In addition, rfc0020 only affects the slow end of adjustment, providing more timestamps that shift the difficulty down only when the time since the last PoW block exceeds the target.
Here, we seek to work on the fast side of the behavior, moving the difficulty up faster when blocks come in before the target.
As such, we will use a piecewise augmentation of the interval for more aggressive upward motion of the difficulty.

To maintain behavior of the exponential moving average, as well as the targeted spacing, we should augment either the timespan or the interval.  As a rough example written by someone who is not a cpp developer, we will augment the locally defined 'nInterval' 
variable with separate definitions for downward PoW difficulty changes.
We use a linear function of the actual spacing to multiply the speed with which the algorithm adjusts the difficulty.

        Interval *= [(1/a)*ActualSpacing/TargetSpacing - (a-1)/a]

Where '*=' means 'multiply the value by'.

We can choose an 'a' that gives us the faster response we desire.
Specifically, a choice of 2 or 4 corresponds to a 2x or 4x modification at ActualSpacing=0, i.e. instant PoW block strings.
As we can sometimes see 6-8 block strings in current operation of the chain, setting a=4 would be a targetted choice to reduce these strings to 1 or 2 blocks.

Defining the augmentation would then appear like this (please forgive this attempt) after the declaration of nInterval:


                if (IsProtocolVThisRFC(pindexLasrt->nTime)) {
                    if (!fProofOfStake){
                        if (nActualSpacing < nTargetSpacing){
                            if (nActualSpacing < 0){
                                nActualSpacing = 0;
                            }
                            nInterval *= ((1/4) + ((3/4) * nActualSpacing/nTargetSpacing));
                        }
                    }
                }

We could also place the calculation of nInterval higher into the preceding 'if' statement that already includes fProofofStake, which we list in the unresolved questions section.

This implementation is continuous (i.e. it limits to 1 at ActualSpacing=TargetSpacing).  It is also differentiable to first order across this limit, making it a 'smooth' function that will provide good participant behaviors.

 Another way of seeing the effect of this proposal is from the perspective of what the difficulty does in a clustered chain vs an evenly distributed chain.
 If there are 2 blocks in 2 times the target time, the difficulty will be constant if they are evenly spaced.
 If they are clustered at 0 and 2 times the target, then the difficulty will increase by ~4% then decrease by ~1%.
 This trend toward higher difficulty for clustered chains is a direct consequence of adjusting the difficulty sharper on the up swing than the down swing, but it also can hopefully have secondary socially-driven effects.
 The game-theoretic behavior of miners that wish to increase their reward for a given hash rate will be to spread it out evenly in time, rather than contributing to a boom or bust cycle.
 This, together with the tighter algorithm in the wake of rfc20, should contribute strongly to a steady stream of work-distributed block rewards.

## Drawbacks

This proposal makes the code more complex.
Specifically and unfortunately, it may incur complications from the implementation of rfc20.
This proposal can also make the difficulty changes less intuitive, which could drive away miners.
On the other hand, it should prevent the boom and bust cycles that could be currently driving miners away.

## Alternatives

We could decrease nInterval by a factor of 2 or 4, making the effective TargetTimespan a couple of days rather than a week.
This would be simpler algorithmically, but would include many of the same 'if' statements.

We could hypothetically do this with any factor or target interval of stabilization.
However, this is just a simple linear tuning knob and will never capture the asymmetric behavior of a pow boom and bust cycle.
Still, tightening this parameter instead of the detailed proposal is a possibility that should be considered post-rfc20.

## Unresolved questions?

Is it better to define nInterval within the if (fProofofStake) loop, after the check for version 0.9, rather than making a separate if statement?
