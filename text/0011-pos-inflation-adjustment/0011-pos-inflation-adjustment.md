# Proof-of-Stake inflation adjustment

- Status: agreed
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
- A '1-year' period is considered to be (365 * 33 + 8)/33 days

## Motivation

Increasing PoS block reward will likely increase minter participation, which is desirable for network security.
In addition, having a clearly defined inflation rate in units of percent of total supply per year is considered healthy economic policy.
As such, it is desirable to increase the PoS block reward until it approaches a target inflation, and to decrease the reward if it exceeds this target.
It is assumed that the Peercoin network has concensus for a PoS inflation of `nRewardCoinYear`, currently 0.01, even when 100% of coins are minting, and so we will seek to produce a protocol modification that increases the PoS reward when less than 100% of coins are participating in the minting process.
We will also seek to reward most those that mint continuously and use the standard splitting protocols defined in the client, as opposed to periodic minters or those that stake grind.

## Detailed design

The function `GetProofOfStakeReward` in the Peercoin code contains the formula for calculating the PoS block reward.
While there may be other locations in the code where this value is referenced or checked (for example incomming blocks) we will limit the design discussion to a simple scaling of this formula.  
The scaling factor thereby multiplies the block reward by a value, which we shall call `nInflationAdjustment`.  
We will define `nInflationAdjustment` to rely on two indices: `nMoneySupply` and `nAnnualStake`.

`nMoneySupply` was well defined in previous versions of Peercoin and should be the sum of all coins currently in existance at the time of block formation.
After each block, `nMoneySupply` is adjusted by adding the coins created in the block reward minus the coins consumed via burn or transaction fees.

`nAnnualStake` will be a sum across all blocks within the past year of the coindays that participated in the staking process, defined as `nCoinAge * 33 / (365 * 33 + 8)`, where `nCoinAge` is limited to a maximum of (365 * 33 + 8)/33 coindays per coin.
At the time of each PoS block, `nAnnualStake` is adjusted by subtracting coindays corresponding to PoS blocks exceeding a year from the timestamp of the block.
After each PoS block, `nAnnualStake` is adjusted by adding the corresponding coindays.

The scaling factor will also rely on the parameter `nAdjustmentMaximum` which we will set at 5. 
The minimum adjustment will be taken as `nRewardCoinYear * 100`.

First, we find the adjustment to the block reward as a number near 1.
> int64 nUnboundedInflationAdjustment = nMoneySupply/nAnnualStake  

Bound the number with minimums and maximums.  The adjustment maximum should be considered a critical parameter.  
> int64 nInflationAdjustment = Maximum[Minimum[nUnboundedInflationAdjustment , nAdjustmentMaximum] , nRewardCoinYear * 100]  

nVariableSubsidy, analogous to nSubsidy, can be modified directly using nInflationAdjustment.
We will use `nVariableWeight` in place of `nRewardCoinYear`, which will be explored in its own section.
> int64 nVariableSubsidy = nCoinAge * 33 / (365 * 33 + 8) * nVariableWeight * nInflationAdjustment;  

*Limiting Coindays*  
The year-long averaging of `nAnnualPoSRewards` implies that a stake-holder might gain an advantage by storing up their coindays for years until they find a favorable `nInflationAdjustment`.
To prevent this, we would impose a 1-year limitation on coindays, such that any output that is held for longer than 1-year will have a maximum coindays of (365 * 33 + 8)/33 coindays per coin.

*Static Block Rewards*  
To promote continuous minting and to punish minters who attempt to time their stake (see 'Timing Attacks' under Drawbacks, specifically with respect to the runaway timing attack), a static block reward will also be added.
Because the inflation adjustment normalizes the variable subsidy to a particular inflation rate per year, we can equally make the static reward proportional to `nMoneySupply` and thereby maintain self-consistency.
We will use an average blocks per day (1440 minutes / `nBlockTarget`, equivalent to 144 given 10 minute blocks) in order to target a particular annual inflation rate with reasonable accuracy.
In order to maintain the target inflation rate of `nRewardCoinYear`, we will relate the static reward to the inverse ratio of the variable reward.
> int64 nStaticSubsidy = (1 - 'nVariableWeight) * nMoneySupply * 33 / (365 * 33 + 8) * nBlockTarget / 1440

*Variable Weight*  
To put everything together, we now have a variable portion and a static portion that, when taken over the course of a full year, sum approximately to `nMoneySupply`.
Therefore, we will create a compound `nSubsidy` that is targeted at a specific inflation rate.
> int64 nSubsidy = nRewardCoinYear * (nVariableSubsidy + nStaticSubsidy)

`nVariableWeight` can be taken as the component of the reward that is related to coinage, while its inverse is static.
This weighting can be seen as a critical parameter that adjusts the relative reward and punishment of different minter behaviors.
We will take `nVariableWeight` = 0.75 and discuss motivation for this choice in the Alternatives section.

Ultimately, with each of these concerns taken in tandem, the ideal behavior of the minter will also be the most heavily rewarded, that of leaving the minting node on and allowing the standard stake splitting process that is already programmed into the wallet software.

## Drawbacks

*Increased Node Complexity due to Two Indices*  
`nAnnualStake` and `nMoneySupply` are both indices that can be acquired as indices during initial download of the blockchain and updated on subsequent blocks.
The memory and processing requirements of maintaining an additional floating constant are likely negligible.
Maintaining these indices from a development perspective is less trivial, though there is precident for storing `nMoneySupply` as an index at least.
Upgrading `nMoneySupply` to an index may make it easier to call that particular parameter, which is a trivial side benefit of this rfc.
Similarly, `nAnnualStake` may also be a statistic of interest for those curious about the participation of minters on the chain.

*Obfuscated Rewards*
It becomes difficult to calculate by hand exactly what a minter is owed because the reward is no longer a straightforward 1%/year on whatever coins are held.
This may be offputting to some minters, though their earned reward should be higher under RFC-0011 in nearly all cases.

*Imprecise nAnnualStake*  
The use of `nCoinAge` to calculate the annual stake may overestimate the true number of coins minting because blocks close to the 1-year expiry will have a coinage greater than the number of coins times the difference between the block stamp and the 1-year expiry date.
Similarly, it may underestimate the true number of coins minting because it is impossible to know how many coins are held across the network in minting wallets at the moment an individual block is created.
`nAnnualStake` is a best attempt to approximate the number of coins minting in the last year on average.
There is no reliable way to game this imprecision aside from a basic Timing Attack, detailed in the following drawback.

*Timing Attacks*  
Minting before finding a valid PoS block is not possible, however a minter may always withhold blocks in an attempt to attain the best reward.
There are two variables that contribute to `nInflationAdjustment` that can be timed.  
The first, `nMoneySupply`, is so insensitive to block-by-block changes that we will ignore its affects as a concern.
This constitutes a generic statement that any `nInflationAdjustment` with a first order dependence on `nMoneySupply` alone is not susceptible to timing attacks.
The second variable that can be timed is `nAnnualStake`, where we will focus for the remainder of this drawback.

To instruct, we shall observe two cases:  
1. Immediately following the implementation, due to the current low level of PoS reward: `nInflationAdjustment = nPoSInflationMaximum`.
In this situation, the minter is receiving a 5x increase of their PoS reward, therefore this situation is ideal to the minter.
The value of `nInflationAdjustment` will not fluctuate from the maximum during this stage, and Timing Attacks are irrelevant as the adjustment cannot be increased above the maximum.

2. If we assume that a consistent fraction of minters participate reliably, then after some time nInflationAdjustment will approach a somewhat stable value.
In this situation, `nInflationAdjustment = 1/(fraction of minters participating)`.
The value of `nInflationAdjustment` will fluctuate on large timescales during this stage.

*Runaway Timing Attack*
When making a Timing Attack, the attacker will seek to move their block rewards closer to case 1.
When choosing to withhold PoS blocks, the attacker is ultimately betting that `nAnnualStake` will decrease with time.
This is most likely to be a good bet when the system is naturally moving from case 2 to case 1, i.e. when a significant number of minters drop out.
Those minters who perceive this happening will then also drop out, hoping for more rewards later, thereby causing a runaway timing attack until the system equilibrates at case 1.
The deviation of `nVariableWeight` from 1, i.e. portion of static reward, will directly counteract such a runaway timing attack by rewarding all those who continue to mint based on a fraction of the total coin supply, rather than the size of the outputs used to mint and regardless of the current value of `nInflationAdjustment`.

*Seasonal Timing Attack and Year-to-Year Timing Attack*  
`nAnnualStake` is taken as a yearly average, so a seasonal timing attack whereby a minter waits for a favorable portion of the year to mint is not relevant.
`nCoinAge` is limited to 1-year per coin, so minters are not able to wait for a favorable year to mint.

*Stake Grinding the Static Reward*
Including a static component in the reward motivates minters to split their outputs down to small amounts in order to maximize the number of blocks they might mint in a year.
The 1 year limitation on coinage, combined with the magnitude of `nVariableWeight`, prevents minters from collecting some of the variable portion of their reward if they split their outputs small enough that they no longer mint within a year's time.
These measures will greatly reduce blockchain bloat when compared to a purely static reward (`nVariableWeight` = 0) where grinding outputs down to their dust values becomes most effective.

*Limited CoinAge*  
Limiting the reward of `nCoinAge` to 1-year per coin is a detriment to small minters who mint very rarely.
Random chance can sometimes cause outputs with nearly 100 coins to take greater than a year to mint, even if a minter's wallet is always online and unlocked for minting.
These outputs will lose potential reward for every day over the 1-year limit during which their coins do not mint.
On the other hand, this limitation will incentivize continuous minting and prevent years-old outputs from claiming a large mint reward that is arguably undeserved.
In addition, the inclusion of a static reward will benefit small minters when they get lucky and find a block.

*Exacerbated Protocol Attacks*  
N@S and Stake Grind attacks are currently considered impractical because the reward to relative cost is very small, being limited only to a small amount of compounding interest.
By increasing the PoS reward, we increase the risk associated with such attacks. It is also possible that an attacker might manipulate `nInflationAdjustment` via a N@S and/or Stake Grind attack.

As a case study, we can imagine a world where an attacker has near-infinite computational resources and perfect knowledge over the network.
We will also assume that they have ~20% of the minting power, and that 50% of the network is minting (such that the attacker posesses 10% of the entire network).
We will then assume that the attacker uses half of their coins to create millions of small stake outputs, and the other half of their coins goes into a single output.
The attacker will then mint a long chain in secret using the small stakes and cap it off with the large stake transaction before broadcasting it.
We want to isolate the effects of this RFC on the attacker reward for forcing a large reorg of this nature, so we will ignore possibilities such as Double Spend attacks which are unaffected by this RFC.
Again, we look to the effects on `nAnnualStake`, which is reduced for every block reorged because the attacker is replacing the staked outputs with a minimal output.
However, we must realize that this effect is averaged over the entire year, similar to a seasonal attack, so if we assume that the PoS rewards are evenly distributed throughout the year aside from the attacker's blocks, then the total effect on the attacker's `nInflationAdjustment` will only be approximately 1/52560 per block of stake grind.
To have a substantial effect, this would mean that the attacker would have to find thousands of blocks in private and still beat the main chain's difficulty, which is unlikely.
In addition, the best result they can hope for is that they go from 2% per year to `nAdjustmentMaximum`, and they will likely lose a considerable chunk of compounding interest doing so.

As such, we should assume that the main influence RFC-0011 has on exacerbating protocol attacks comes strictly from the increased block reward, rather than direct manipulation over `nInflationAdjustment`.
Therefore, to approximate the additional risk introduced to the protocol via N@S or stake grind attacks, one should simply assume that RFC-0011 will result in the protocol continuously operating at `nAdjustmentMaximum`.

*Impact on Futuristic Signers*  
It is currently possible to form and sign a coinstake many blocks in advance of inclusion in a block.
One example of a use case for this would be multisig minting.
As prediction of the precise `nSubsidy` is effectively impossible far in advance, this proposal could either disallow such action or require the minter(s) to use an `nSubsidy` sufficiently smaller than what is allowed.
In the most stringent application of the latter possibility, the minter(s) should default to the minimum adjustment to ensure inclusion in the chain.

## Alternatives

*Choice of nVariableWeight*
The weighting of the variable portion with respect to the static portion should be seen as a balancing act.
On the one hand, a weight of 1 would benefit those that perform timing attacks and could result in a runaway timing attack.
On the other hand, a weight of 0 would benefit those that stake grind down to dust values.
In order to see why a weighting of 0.75 is ideal, we can first analyze the case of `nVariableWeight` = 0.5, i.e. a 50/50 split between variable and static portions of the reward.

A long-term timing attack would only beneficial to the minter if they recover the potential static rewards they could have had through a fluctuation in the adjustment.
Because the static rewards are given only to those actively minting, they can be seen as proportional to the minter participation, similar to the variable reward but without a moving average.
With a weight of 0.5, the timing attacker only breaks even if the adjustment moves by 100%, in order to double the amount made on the variable portion and make up for the amount lost on the static portion.
However, it should be noted that this 100% is taken from the perspective of the true participation, rather than the current inflation adjustment, and so the minter would need the inflation adjustment to move 100% of the way to its new value.
This is impossible because there is a minimum inflation adjustment, and so with a weight of 0.5 a long-term timing attack is always less profitable than a stake grind attack.
For this reason, the long-term timing attack can never justify a reduction of `nVariableWeight` below 0.5.

On the other hand, a weighting of 0.5 appears to profit the stake grinder greatly.
Given the 1 year limitation on coinage, the stake grinder is able to grind their stake down to the point where they are likely to mint within as much as 2 years and still make back the fraction lost in coinage.
Given this, it seems prudent to increase `nVariableWeight` above 0.5.

With this understanding, we can now establish the tradeoff that `nVariableWeight` represents.
A weighting of 0.67 would require a timing attack on the order of a 50% change in inflation adjustment (based on the higher end), for example from 2 to 4.
On the other hand, it would allow for a grinding down to 6 months past the 1 year mark for stake.

The value of 0.75 is chosen because a 33% change in inflation adjustment (for example, from 3 to 4.5) seems to be on the edge of acceptable.
Similarly, 4 months past the 1 year coinage also appears to be an acceptable time frame.

Other values, such as 0.8 for a 25% change in adjustment to benefit from timing attacks versus 3 months past 1 year to benefit from stake splitting, are also possible.
However, the choice of 0.75 appears to be an appropriate balance amongst these options.

*Difficulty/Dayweight Implementation*  
In order to alleviate the potential for a runaway timing attack related to the slow moving average of 'nAnnualStake' compared to the PoS Difficutly, we could replace 'nAnnualStake' in the computation of 'nUnboundedInflationAdjustment' with 'nMintingCoins'.
In calculating 'nMintingCoins', it is noted that the average number of coins minting can be estimated from the PoSDifficulty.
This is done by realizing that the protocol allows for 1 hash per second, that there are 2^256 possible hashes, and that the target is defined as 2^224/difficulty.
Therefore, the number of coins minting can be estimated as:

> coins = (PoSDifficulty * 2^32) / (dayweight * 600)

The 600 is a reference to the 10-minute block target, taken as 'nTargetTime'.
The year-long average over 'nAnnualStake' will be replaced with a year-long average over dayweight, named 'nAnnualDayWeight'.
The 'nDifficulty' is taken as the PoS difficulty target of the current block.
Therefore, leaving the rest of the proposal the same, the adjustment can be calculated as follows:

> int64 nUnboundedInflationAdjustment = nMoneySupply * nAnnualDayWeight * nTargetTime / (nDifficulty * 2^32)

The benefits of this implementation are that the adjustment will increase directly when the difficulty languishes, stimulating minters to participate exactly at the time when the chain is most vulnerable.
The average 1%/year would still be approximately followed for the same reasons that the original implementation laid out, though the deviation from this number would be stronger under this implementation.
The drawbacks would be that the most effective attacks on adjustment would come from those manipulating the PoS difficulty, which is the hallmark of network security in Peercoin.
One such attack is analgous to the timing attack, but on a much shorter time scale.
When the PoS difficulty is dropping, a minter may choose to withhold their stake until the difficulty has reached a certain low value to maximize their adjustment.
However, as the difficulty is much more drastically affected by a single block than 'nAnnualStake', the very next minter will gain an increased reward while the following minters will not receive such a benefit.
This will prevent minters from performing such an action unless they control an exceedingly large portion of the minting coins, as they cannot depend on others to withhold their stake in the short term.
Taken together with the maximum adjustment, a runaway form of this attack will not occur in the Difficulty/Dayweight Implementation, though the generic timing attack using predictions of minting times is still feasible.

The overarching issue with a difficulty-based implementation is that the difficulty is by design an equilibrium-seeking parameter.
This means that it is regularly out of equilibrium with the true minting participation in the system.
Ultimately, this form of implementation cannot not accurately measure the total coins minting, and indeed has been shown to greatly underestimate the participation.
As a result, an implementation of this sort would drastically overestimate the required inflation adjustment and will result in significantly greater than 1% inflation.
In addition, it is fairly easy to manipulate the rules that are designed for stabilizing blocktime and not inflation rate.

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
Let us define `nAnnualStake`/`nMoneySupply` as x, representing network participation, and `nUnboundedInflationAdjustment` as f(x), representing the functional form.
For all x between 0 and 1, we require that xf(x) is always less than or equal to 1, in order to avoid increasing the total inflation beyond the inflation target of `nRewardCoinYear`, and f(x) must be positive.
By mandating xf(x)=1, we can easily see that f(x)=1/x is the only answer.
We can find alternate forms, such as f(x)=1-ln(x), where ln(x) is the natural logarithm of x.
A functional form of this type will be part way between the current protocol, where f(x)=1, and the given proposal, where f(x)=1/x, in that it will result in less than 1% total inflation for all participations less than 100%, but will still award more mint rewards than the current protocol.

## Unresolved questions

What parts of the design are still to be done?
