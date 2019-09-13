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

*Recursive Calculation of nSubsidy*  
If a very large output is minted, it could get an unfair portion of the annual reward before `nInflationAdjustment` is able to compensate for it.
In order to avoid this, the calculation of `nAnnualPoSRewards` should include the block that is being minted.
This requires a recursive calculation, such that `nSubsidy` is first calculated using the rewards of the past 365 days.
Next, `nSubsidy` is calculated again with this previous calculation of `nSubsidy` added in to `nAnnualPoSRewards`.
Further recursive calculation could allow a more accurate convergence on the appropriate `nSubsidy`, but for simplicity it is sufficient to maintain a recursion limit of 1.
The result will underestimate `nSubsidy`, thereby discouraging large single output minting.
Single outputs that do not represent a significant portion of `nMoneySupply` will not be affected significantly.

## Drawbacks

*Negligible Increased Load on Nodes*  
`nAnnualPoSRewards` is a sum over PoS rewards in the last year which can be acquired as an index when downloading the blockchain and subsequent blocks.
The memory and processing requirements of maintaining an additional floating constant are negligible.
Similarly, by limiting the calculation of `nSubsidy` to a single recursion, the additional load on minters and validators to calculate `nSubsidy` is negligible.

*Impact on Futuristic Signers*  
It is currently possible to form and sign a coinstake many blocks in advance of inclusion in a block.
One example of a use case for this would be multisig minting.
As prediction of the precise `nSubsidy` is effectively impossible far in advance, this proposal could either disallow such action or require the minter(s) to use an `nSubsidy` sufficiently smaller than what is allowed.
In the most stringent application of the latter possibility, the minter(s) should use `nAdjustmentMinimum` in their calculation.

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
In this situation, `nInflationAdjustment` will quickly sink to below 1 and remain there.
The equilibrium value will be driven to approximately 0.99 due to compounding interest.
The value of `nInflationAdjustment` will fluctuate minimally above and below the equilibrium value due to statistical aberration and random coin distribution.

When making a Timing Attack, the attacker will seek to move their block rewards closer to case 1.
When choosing to withhold PoS blocks, the attacker is ultimately betting that `nAnnualPoSRewards` will decrease with time.
This may be a reliable bet during specific moments when year old blocks with a large PoS reward age out.
However, it is very difficult to guess how many coins will be used to stake in the near future.
Therefore, it is nearly always in the Timing Attacker's best interest to release their PoS block immediately in order to attain the highest compounding interest. 

*Seasonal Timing Attack*  
Some minters greatly limit their minting intervals in order to avoid security vulnerabilities associated with having a node online.
Let us assume that a 'seasonal minter' of this fashion will only mint, for example, twice a year.
This minter is incentivized to analyze the seasonal fluctuations of `nInflationAdjustment` and pick the two times during the year when minter participation is lowest.
This type of minting will act to dampen the aforementioned seasonal fluctuations of `nInflationAdjustment`, which will help to reduce the standard deviation of the PoS difficulty from the mean.
As the PoS model of security is only as strong as its moment of lowest PoS difficulty (not the mean), this type of behavior (that of actively attempting to mint during periods of low minter participation) should be encouraged for those that insist on being seasonal minters.
As such, the 'seasonal timing attack' is beneficial to the security of the Peercoin chain, rather than a detriment.

*Timestamp Attack*  
A minter has 2 hours to place a block on the chain.
A minter can also somewhat reliably predict the window for minting an output as much 30 days in advance.
If the minter finds two blocks within the shorter time window, the later block will always have an equal or lower `nAnnualPosRewards`.
If the minter finds two blocks within the longer time window, they can choose to withhold their first block as a Timing Attack with a high probability of success.
Both of these forms encourage minters to use later timestamps, which is a mild but undesireable side effect.

*Exacerbated Protocol Attacks*  
N@S and Stake Grind attacks are currently considered impractical because the reward to relative cost is very small, being limited only to a small amount of compounding interest.
By increasing the PoS reward, we increase the risk associated with such attacks. It is also possible that an attacker might manipulate `nInflationAdjustment` via a N@S and/or Stake Grind attack.

As a case study, we can imagine a world where an attacker has near-infinite computational resources and perfect knowledge over the network.
We will also assume that they have ~20% of the minting power, and that 50% of the network is minting (such that the attacker posesses 10% of the entire network).
We will then assume that the attacker uses half of their coins to create millions of small stake outputs, and the other half of their coins goes into a single output.
The attacker will then mint a long chain in secret using the small stakes and cap it off with the large stake transaction before broadcasting it.
We want to isolate the effects of this RFC on the attacker reward for forcing a large reorg of this nature, so we will ignore possibilities such as Double Spend attacks which are unaffected by this RFC.
Again, we look to the effects on `nAnnualPoSRewards`, which is reduced for every block reorged because the attacker is taking everyone else's PoS rewards.
However, we must realize that this effect is averaged over the entire year, so if we assume that the PoS rewards are evenly distributed throughout the year aside from the attacker's blocks, then the total effect on the attacker's `nInflationAdjustment` will only be approximately 1/52560 per block of stake grind.
To have a substantial effect, this would mean that the attacker would have to find thousands of blocks in private and still beat the main chain's difficulty, which is unlikely.
In addition, the best result they can hope for is that they go from 2% per year to `nAdjustmentMaximum`, and they will likely lose a considerable chunk of compounding interest doing so.

As such, we should assume that the main influence RFC-0011 has on exacerbating protocol attacks comes strictly from the increased block reward, rather than direct manipulation over `nInflationAdjustment`.
Therefore, to approximate the additional risk introduced to the protocol via N@S or stake grind attacks, one should simply assume that RFC-0011 will result in the protocol continuously operating at `nAdjustmentMaximum`.

## Alternatives

*Year by Block*  
In order to invalidate the Timestamp Attack, `nAnnualPoSRewards` could sum over a number of blocks rather than a unit of time.
However, this alternative is more susceptible to the Stake Grind attack, wherein the attacker will intentionally starve the Inflation rate using many low value coinstakes, then mint rapidly with high value coinstakes at maximum inflation.
This is because coindaysdestroyed/block is sensitive to the Stake Grind while coindaysdestroyed/year is not. 

*Fixed Reward*  
Many coins are going to a model where each block gives a fixed reward (such as 5 PPC/block).
This is generically untenable with the Peercoin PoS method, as it would remove coindays from the calculation.
This would tilt the relative reward in favor of using small outputs, greatly benefitting the Stake Grind Attacker.

*Generalized Functional Form*  
One could imagine any number of alternative functions to use when calculating `nInflationAdjustment`.
However, a series of stringent criteria ultimately limit the possible choices.
Let us define `nAnnualPoSRewards`/`nMoneySupply` as x, representing network participation, and `nUnboundedInflationAdjustment`/`nPoSInflationTarget` as f(x), representing the functional form.
For all x between 0 and 1, we require that xf(x) is always less than or equal to 1, in order to avoid increasing the total inflation beyond the inflation target, and f(x) must be positive.
By mandating xf(x)=1, we can easily see that f(x)=1/x is the only answer.
We can find alternate forms, such as f(x)=1-ln(x), where ln(x) is the natural logarithm of x.
A functional form of this type will be part way between the current protocol, where f(x)=1, and the given proposal, where f(x)=1/x, in that it will result in less than 1% total inflation for all participations less than 100%, but will still award more mint rewards than the current protocol.

## Unresolved questions

What parts of the design are still to be done?
