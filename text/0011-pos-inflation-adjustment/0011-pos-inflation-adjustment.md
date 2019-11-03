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
- A '1-year' period is considered to be (365 * 33 + 8)/33 days

## Motivation

Increasing PoS block reward will likely increase minter participation, which is desirable for network security.
In addition, having a clearly defined inflation rate in units of percent of total supply per year is considered healthy economic policy.
As such, it is desirable to increase the PoS block reward until it approaches a target inflation, and to decrease the reward if it exceeds this target.
It is assumed that the Peercoin network has concensus for a PoS inflation of `nRewardCoinYear`, currently 0.01, even when 100% of coins are minting, and so we will seek to produce a protocol modification that increases the PoS reward when less than 100% of coins are participating in the minting process.

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

nSubsidy can be modified directly using nInflationAdjustment.  
> int64 nSubsidy = nCoinAge * 33 / (365 * 33 + 8) * nRewardCoinYear * nInflationAdjustment;  

*Limiting Coindays*
The year-long averaging of `nAnnualPoSRewards` implies that a stake-holder might gain an advantage by storing up their coindays for years until they find a favorable `nInflationAdjustment`.
To prevent this, we would impose a 1-year limitation on coindays, such that any output that is held for longer than 1-year will have a maximum coindays of (365 * 33 + 8)/33 coindays per coin.

## Drawbacks

*Increased Complexity due to Two Indices*  
`nAnnualStake` and `nMoneySupply` are both indices that can be acquired as indices during initial download of the blockchain and updated on subsequent blocks.
The memory and processing requirements of maintaining an additional floating constant are likely negligible.
Maintaining these indices from a development perspective is less trivial, though there is precident for storing `nMoneySupply` as an index at least.
Upgrading `nMoneySupply` to an index may make it easier to call that particular parameter, which is a trivial side benefit of this rfc.
Similarly, `nAnnualStake` may also be a statistic of interest for those curious about the participation of minters on the chain.

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
The value of `nInflationAdjustment` will fluctuate regularly during this stage.

When making a Timing Attack, the attacker will seek to move their block rewards closer to case 1.
When choosing to withhold PoS blocks, the attacker is ultimately betting that `nAnnualStake` will decrease with time.
This may be a reliable bet during specific moments when year old blocks with a large stake age out.
However, it is very difficult to guess how much coinage will be used to stake in the near future.
Therefore, it is nearly always in the Timing Attacker's best interest to release their PoS block immediately in order to attain the highest compounding interest. 

*Seasonal Timing Attack and Year-to-Year Timing Attack*  
`nAnnualStake` is taken as a yearly average, so a seasonal timing attack whereby a minter waits for a favorable portion of the year to mint is not relevant.
`nCoinAge` is limited to 1-year per coin, so minters are not able to wait for a favorable year to mint.

*Limited CoinAge*
Limiting the reward of `nCoinAge` to 1-year per coin is a detriment to small minters who mint very rarely.
Random chance can sometimes cause outputs with nearly 100 coins to take greater than a year to mint, even if a minter's wallet is always online and unlocked for minting.
These outputs will lose potential reward for every day over the 1-year limit during which their coins do not mint.
On the other hand, this limitation will incentivize continuous minting and prevent years-old outputs from claiming a large mint reward that is arguably undeserved.

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
In the most stringent application of the latter possibility, the minter(s) should use `nRewardCoinYear * 100` in their calculation.

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
Let us define `nAnnualStake`/`nMoneySupply` as x, representing network participation, and `nUnboundedInflationAdjustment` as f(x), representing the functional form.
For all x between 0 and 1, we require that xf(x) is always less than or equal to 1, in order to avoid increasing the total inflation beyond the inflation target of `nRewardCoinYear`, and f(x) must be positive.
By mandating xf(x)=1, we can easily see that f(x)=1/x is the only answer.
We can find alternate forms, such as f(x)=1-ln(x), where ln(x) is the natural logarithm of x.
A functional form of this type will be part way between the current protocol, where f(x)=1, and the given proposal, where f(x)=1/x, in that it will result in less than 1% total inflation for all participations less than 100%, but will still award more mint rewards than the current protocol.

## Unresolved questions

What parts of the design are still to be done?
