# Proof-of-Stake inflation adjustment

- Status: proposed
- Type: protocol adjustment
- Related components: 
- Start Date: 08-11-2018
- Discussion: https://github.com/peercoin/rfcs/issues/19
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: Nagalim

## Summary

While PoS inflation can currently be up to 1% per year, in reality only a fraction of the existing coins participate in the minting process.
This RFC will seek to continuously adjust the PoS block reward such that PoS inflation approximates to 1%/year on average, no matter how much PoS participation there is.
In general, this RFC will look into the intricacies of tying a block reward to the total supply, as well as how to safely implement a fluctuating block reward.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- An 'Attack' is generically any action performed in an attempt to gain an unfair advantage when using the Peercoin protocol.
- A 'Nothing at Stake Attack' (N@S for short) is defined as any minter that intentionally mints a block on a chain that is not the longest valid chain.
- A 'Stake Grind Attack' is defined as any minter that intentionally splits his/her inputs in order to alter the odds of minting a block.

## Motivation

Increasing PoS block reward will likely increase minter participation, which is desirable for network security.
In addition, having a clearly defined inflation rate in units of percent of total supply per year is considered healthy economic policy.
As such, it is desirable to increase the PoS block reward until it approaches a target inflation, and to decrease the reward if it exceeds this target.

## Detailed design

The function `GetProofOfStakeReward` in main.cpp of the Peercoin code contains the formula for calculating the PoS block reward.
While there may be other locations in the code where this value is referenced or checked (for example incomming blocks) we will limit the design discussion to a simple scaling of this formula.  
The scaling factor thereby multiplies the block reward by a value, which we shall call `nInflationAdjustment`.  
We will define `nInflationAdjustment` to rely on nMoneySupply and a factor we shall call `nAnnualPoSRewards`.  
It will also rely on three parameters, the first of which, `nPoSInflationTarget`, we will set at 0.01.  
`nAdjustmentMaximum` we will set at 5 and `nAdjustmentMinimum` we will set at 0.1.  
`nAnnualPosRewards` is the integrated sum of all PoS block rewards in the last 365 days (by timestamp).

First, we find the adjustment to the block reward as a number near 1.
> int64 nUnboundedInflationAdjustment = nPoSInflationTarget * nMoneySupply/nAnnualPoSRewards  

Bound the number with minimums and maximums.  The adjustment maximum should be considered a critical parameter.  
> int64 nInflationAdjustment = Maximum[Minimum[nUnboundedInflationAdjustment , nAdjustmentMaximum] , nAdjustmentMinimum]  

nSubsidy can be modified directly using nInflationAdjustment.  
> int64 nSubsidy = nCoinAge * 33 / (365 * 33 + 8) * nRewardCoinYear * nInflationAdjustment;  

## Drawbacks

*Increased Load on Nodes*  
`nAnnualPoSRewards` is a sum over PoS rewards in the last year.
Creating and validating blocks will require a search through a substantial number of most recent block (~50,000) PoS rewards which increases the load on both minting and nonminting nodes.
A node with recent blocks still in memory, such as during a fresh download of the blockchain, may be able to avoid the increased load.

*Exacerbated Protocol Attacks*  
N@S and Stake Grind attacks are currently considered impractical because the reward to relative cost is very small, being limited only to a small amount of compounding interest.
By increasing the PoS reward, we increase the risk associated with such attacks.

*Timing Attacks*  
Minting before finding a valid PoS block is not possible, however a minter may always withhold blocks in an attempt to attain the best reward.
There are two variables that contribute to `nInflationAdjustment` that can be timed.  
The first, `nMoneySupply`, is so insensitive to block-by-block changes that we will ignore its affects as a concern.
This constitutes a generic statement that any `nInflationAdjustment` with a first order dependence on `nMoneySupply` alone is not susceptible to timing attacks.
The second variable that can be timed is `nAnnualPoSRewards`, where we will focus for the remainder of this drawback.

To instruct, we shall observe three extreme cases:  
1. Immediately following the implementation, due to the current low level of PoS reward: `nInflationAdjustment = nPoSInflationMaximum`.
In this situation, the minter is receiving a 5x increase of their PoS reward, therefore this situation is ideal.
The value of `nInflationAdjustment` will not fluctuate from the maximum during this stage.

2. If we assume that a consistent fraction of minters participate reliably, then after some time nInflationAdjustment will approach a somewhat stable value.
In this situation, `nInflationAdjustment = 1/(fraction of minters participating)`.
The value of `nInflationAdjustment` will fluctuate regularly during this stage.

3. Suddenly, every single minter begins minting consistently and continuously for the rest of time.
In this situation, `nInflationAdjustment` will quickly sink to 1 and remain there.
The value of `nInflationAdjustment` will fluctuate minimally, both above and below 1, due to statistical aberration and random coin distribution.

When making a Timing Attack, the attacker will seek to move their block rewards closer to case 1.
When choosing to withhold PoS blocks, the attacker is ultimately betting that `nAnnualPoSRewards` will decrease with time.
This may be a reliable bet during specific moments when year old blocks with a large PoS reward age out.
However, it is very difficult to guess how many coins will be used to stake in the near future.
Therefore, it is nearly always in the Timing Attacker's best interest to release their PoS block immediately in order to attain the highest compounding interest. 

*Timestamp Attack*  
A minter has 2 hours to place a block on the chain.
A minter can also somewhat reliably predict the window for minting an output as much 30 days in advance.
If the minter finds two blocks within the shorter time window, the later block will always have an equal or lower `nAnnualPosRewards`.
If the minter finds two blocks within the longer time window, they can choose to withhold their first block as a Timing Attack with a high probability of success.
Both of these forms encourage minters to use later timestamps, which is a mild but undesireable side effect.

## Alternatives

*Year by Block*  
In order to invalidate the Timestamp Attack, `nAnnualPoSRewards` could sum over a number of blocks rather than a unit of time.
However, this alternative is more susceptible to the Stake Grind attack, wherein the attacker will intentionally starve the Inflation rate using many low value coinstakes, then mint rapidly with high value coinstakes at maximum inflation.
This is because coindaysdestroyed/block is sensitive to the Stake Grind while coindaysdestroyed/year is not. 

*Fixed Reward*  
Many coins are going to a model where each block gives a fixed reward (such as 5 PPC/block).
This is generically untenable with the Peercoin PoS method, as it would remove coindays from the calculation.
This would tilt the relative reward in favor of using small outputs, greatly benefitting the Stake Grind Attacker.

## Unresolved questions

What parts of the design are still to be done?
