# PoW Spacing Augmentation

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

This form of the equation is a unique one because it satisfies a particular requirement.
If there are n blocks in a time period t, then it doesn't matter when the blocks take place, the equation will result in the same ending difficulty of the chain.
Any form of nNew being linearly related to nActualSpacing will have this same result because of the commutative rule of multiplication.
This behavior is very important for chains that use difficulty as a form of chain weight, because it prevents strings of blocks from gaining weight over even block distributions.
The chainweight in Peercoin, however, is not a function of PoW difficulty at all.
This is a degree of freedom we can leverage to accomplish the goal of more even block distribution.

One way to view the question of what functional form to take is to realize that across a single block the difficulty can move infinitely far down (in the case of eternity between blocks) but can only move up as quickly as the limit of nActualSpacing=0, which is around a 1% change per block.
The variable nInterval is the expected number of blocks in a week, or 168.
The function in the code can then be written as:  New Difficulty = Old Difficulty divided by f(x), where 'x' is the ratio of actual spacing to target spacing.

        f(x) = (167+2*x)/169

In moving away from linearity, we have a wide array of mathematical processes we could choose from.
However, we have a few desired requirements.
First, we want to balance the long term and the short term behavior such that strings of blocks, especially very close together blocks, are penalized more.
Second, we want to somewhat preserve the behavior at long time scales, though a slight increase increase in slope would be fine.
Third we want the game theoretic best practice to coincide with regularly spaced blocks.
Fourth, we want the equation to be continuous and differentiable to avoid any strange edge cases.
And fifth goes without saying, that the algorithm should seek to converge on the target by setting f(1)=1, f(<1)<1, and f(>1)>1.

With the concept of making 0 limit to the same as infinity, we compare the functions x and 1/x.
We seek a combination of these terms that achieves the above requirements.
In so doing, we augment the original spacing parameterization f'(x)=f(g(x)) where g(x)~x+1/x-1=x+(x-1)/x.
However, we would limit this so as to avoid an infinity at x=0.
In choosing the limit, we aim to augment the 0-time increase of difficulty to be a little over 3x the speed of difficulty decrease.
As such, our equation will be:

        g(x)=x+(x-1)/(x+0.3)

Remembering that x=nActualSpacing/nTargetSpacing allows us to see how the code would look.
We will define a new spacing:

        nAugmentSpacing = nTargetSpacing
        nAugmentSpacing *= nActualSpacing - nTargetSpacing
        nAugmentSpacing /= nActualSpacing + nSpacingParameter*nTargetSpacing

And we keep the update function the same, adding on the augmentation the same way nActualSpacing is added:

        bnNew *= ((nInterval - 1) * nTargetSpacing + nActualSpacing + nActualSpacing + nAugmentSpacing + nAugmentSpacing);
        bnNew /= ((nInterval + 1) * nTargetSpacing);

 Another way of seeing the effect of this proposal is from the perspective of what the difficulty does in a clustered chain vs an evenly distributed chain.
 If there are 2 blocks in 2 times the target time, the difficulty will be constant if they are evenly spaced.
 If they are clustered at 0 and 2 times the target, then the difficulty will increase by ~5% then decrease by ~1.6%.
 This trend toward higher difficulty for clustered chains is a direct consequence of adjusting the difficulty sharper on the up swing than the down swing, but it also can hopefully have secondary socially-driven effects.
 The game-theoretic behavior of miners that wish to increase their reward for a given hash rate will be to spread it out evenly in time, rather than contributing to a boom or bust cycle.
 This, together with the tighter algorithm in the wake of rfc20, should contribute strongly to a steady stream of work-distributed block rewards.

## Drawbacks

This proposal makes the code more complex.
Specifically and unfortunately, it may incur complications from the implementation of rfc20.
This proposal can also make the difficulty changes less intuitive, which could drive away miners.

## Alternatives

We could simply change nInterval to adjust the slope of the equation.

        int64_t nInterval = params.nTargetTimespan / (2 * nTargetSpacing);
        
nTargetTimespan from which nInterval is derived represents a week on average in Peercoin v0.1 and this would tune it down to 3.5 days.
We could hypothetically do this with any fraction or target interval of stabilization.
However, this is just a simple linear tuning knob and will never capture the asymmetric behavior of a pow boom and bust cycle.
Still, tightening this parameter instead of the detailed proposal is a possibility that should be considered post-rfc20.

## Unresolved questions?
