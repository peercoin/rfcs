 PoW Spacing Augmentation

- Status: proposed
- Type: protocol adjustment
- Related components: 
- Start Date: 09-23-2022
- Discussion: https://talk.peercoin.net/t/rfc-0024-pow-spacing-augmentation/15894
- Supercedes: (fill me in with a link to RFC this supersedes - if applicable)
- Requisite: rfc-0020
- Author: Nagalim

## Summary

The role of Proof of Work in the Peercoin chain is somewhat unique in its design.
Specifically, the purpose is to provably distribute coin rather than to validate an immutable record.
In this regard, even distribution of blocks that do not disrupt finality of the chain are more important than a guarenteed target average block (and reward) frequency.
With the acceptance of rfc0020 regarding updating the PoW difficulty on PoS blocks, the protocol has the opportunity to tighten up difficulty changes without fearing the backlash of an inescapable difficulty bomb.
With this in mind, we revisit the difficulty adjustment algorithm and look for opportunities to achieve a more stable chain with regular distribution.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Blockchain stability and the concept of finality rely on a regular distribution of blocks.
Long strings of PoW chains can be orphaned with just a single PoS block, damaging both the integrity of the chain and the miners who worked for the blocks.
With this in mind, we seek methods of discouraging multiple PoW blocks in a row.
At the same time, we do not want to affect the rewards and behaviors of the PoS mechanism, so we will not couple any portion of the PoW algorithm to awareness of PoS blocks, aside from the safe hallmarking of time as in rfc0020.
Rather, we will use the passage of time as our sole feedback control and we will identify an appropriate dilation of the difficulty adjustment interval to accentuate difficulty spikes on tightly packed PoW blocks.

More generally, rfc0020 causes the difficulty adjustment to function in a hypersensory state, where it has information from PoW and PoS blocks while only performing the exponential difficulty adjustment on PoW blocks.
We will discuss changes to the difficulty adjustment interval for both the specific case of PoW strings as well as the general case of all PoW blocks.
The additional blocktime data granted by PoS blocks through rfc0020 provides leeway to lower the interval for PoW blocks, resulting in faster difficulty changes.

## Detailed design

The current algorithm for modifying the proof of work difficulty is given in pow.cpp as follows:

        int64_t nInterval = params.nTargetTimespan / nTargetSpacing;
        bnNew *= ((nInterval - 1) * nTargetSpacing + nActualSpacing + nActualSpacing);
        bnNew /= ((nInterval + 1) * nTargetSpacing);

This is an exponential moving average with an nInterval targeting a 1 week timespan (normalized by the target spacing of 1 hour).
The subroutine defined in pow.cpp is common between PoW and PoS difficulty adjustments, albeit with a different target spacing.
This target spacing is set separately for PoS and PoW difficulty adjustments just before the algorithm via 'if (fproofofstake)'.
The 'timespan' is chosen to be a week as a global parameter throughout the code.
Having a simple determination of the timespan and the target, and therefore 'nInterval', is important for chains that use difficulty as a form of chainweight.
The chainweight in Peercoin, however, is not a function of PoW difficulty at all.
This is a degree of freedom we can leverage to accomplish the goal of more even block distribution.

In considering concepts of tuning the difficulty adjustment algorithm, we consider the natural asymmetry of the adjustment.
Across a single block, the difficulty can move infinitely far down (in the case of eternity between blocks) but can only move up as quickly as the limit of nActualSpacing=0, which is around a 1% change per block.
In addition, rfc0020 only affects the slow end of adjustment, providing more timestamps that shift the difficulty down only when the time since the last PoW block exceeds the target.
Fundamentally, we are focused on strings of blocks which are on the opposite end of the time scale, the 'fast' side.

There is an expectation that rfc0020 will even out gaps of many hours between blocks and 'diffuse the difficulty bomb' resulting from difficulty that has been driven up higher than the sustainable hash rate.
If we model the miners as an amorphous group with a target difficulty at which mining is profitable, we can assume that a large portion of miners will redirect their hash rate away from Peercoin when the difficulty over-corrects.
With rfc0020, it is only a matter of time and PoS blocks before the difficulty lowers back toward the mean.
As such, this proposal is not intended to primarily affe on the long average timescale, as that is covered by rfc0020.
Rather, this proposal is aimed at preventing large changes in the network, such as the additional hash power resulting from a rapid price change in the market, from causing frequent pow strings while the difficulty rises to equilibrium.

Toward this end, we move the difficulty faster when blocks come in before the target.
As such, we will use a piecewise augmentation of the interval for more aggressive normalization of the difficulty only when tightening.
To maintain behavior of the exponential moving average, as well as the targeted spacing, we should augment either the timespan or the interval.
As an example, we will augment the locally defined 'nInterval' variable with separate definitions for PoW difficulty changes in one direction.
We use a linear function of the actual spacing to multiply the speed with which the algorithm adjusts the difficulty.

        Interval *= [((a-1)/a)*ActualSpacing/TargetSpacing + 1/a]

Where '*=' means 'multiply the value by'.

We can choose an 'a' that gives us the faster response we desire.
Specifically, a choice of 2 or 4 corresponds to a 2x or 4x modification at ActualSpacing=0, i.e. instant PoW block strings.
Empirically, we sometimes see 6-8 block strings in current operation of the chain.
Therefore, a 4x modification (e.g. a=4) would be a targetted choice to reduce these strings to 1 or 2 blocks.

In addition to the augmented interval, we will also globally decrease the interval for all PoW blocks.
The argumentation for tightening the feedback loop follows from the hypersensory nature of rfc0020.
Toward this end, we look to the PoS/PoW block ratio as an indicator.
A factor of 2x is prudent, simple, and well within the window of 6 given by the block ratio.
By dividing the interval in half on top of an a=2 protocol, we achieve the 4x target mentioned for the desired empirical results.

Defining the augmentation would then appear like this (with the caveat that the rfc writer is not a cpp developer) after the declaration of nInterval:

        int64_t nInterval = params.nTargetTimespan / nTargetSpacing;
        if (IsProtocolVThisRFC(pindexLast->nTime)) {
            if (!fProofOfStake) {
                nInterval /= 2
                if (nActualSpacing < nTargetSpacing) {
                    if (nActualSpacing > 0) {
                        nInterval *= (1/2)*(1 + nActualSpacing/nTargetSpacing);
                    } else {
                        nInterval /= 2;
                    }
                }
            }
        }

We could also place the calculation of nInterval higher into the preceding 'if' statement that already includes fProofofStake, which we list in the unresolved questions section.

This implementation is continuous (i.e. it limits to 1 at ActualSpacing=TargetSpacing).  It is also differentiable to first order across this limit, making it a 'smooth' function that will provide good participant behaviors.

 Another way of seeing the effect of this proposal is from the perspective of what the difficulty does in a clustered chain vs an evenly distributed chain.
 If there are 2 blocks in 2 times the target time, the difficulty will be constant if they are evenly spaced.
 If they are clustered at 0 and 2 times the target, then the difficulty will increase by ~4% then decrease by ~2%.
 This trend toward higher difficulty for clustered chains is a direct consequence of adjusting the difficulty sharper on the up swing than the down swing, but it also can hopefully have secondary socially-driven effects.
 The game-theoretic behavior of miners that wish to increase their reward for a given hash rate will be to spread it out evenly in time, rather than contributing to a boom or bust cycle.
 This, together with the tighter algorithm in the wake of rfc20, should contribute strongly to a steady stream of work-distributed block rewards.

## Drawbacks

This proposal adds several lines of code, and thus requires maintenance if pulling from other codebases.
While it could incur complications from the implementation of rfc0020, it acts exclusively on the 'actual < target' regime while rfc0020 acts on the 'actual > target' regime, so seamless integration should be achievable.
This proposal can also make the difficulty changes less intuitive, which could drive away miners.
On the other hand, it should help prevent the boom and bust cycles that could be currently driving miners away.

## Alternatives

We could just decrease nInterval by a factor of 2 or 4, making the effective TargetTimespan a couple of days rather than a week.
This would be simpler algorithmically, but would include many of the same 'if' statements.
We could hypothetically do this with any factor or target interval of stabilization.
However, this is just a simple linear tuning knob and will never capture the asymmetric behavior of a PoW boom and bust cycle.
Still, tightening this parameter instead of the detailed proposal is a possibility that should be considered post-rfc20.

## Unresolved questions?

Is it better to define nInterval within the if (fProofofStake) loop, after the check for version 0.9, rather than making a separate if statement?
Moving the augmentation up would require pulling up the definition of nInterval itself, which would get messy because it would have to occur concurrently with the definition of TargetSpacing.
