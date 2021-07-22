# Split and Combine

- Status: proposed
- Type: parameter modification, new feature
- Related components: (if any)
- Start Date: 14-March-2020
- Discussion: https://talk.peercoin.net/t/reducing-split-frequency-from-90-days-to-60-days/10125  &&  https://talk.peercoin.net/t/pooling-coinstakes/10124
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: Nagalim

## Summary

The 'sweet spot' for minting an output is between 90 days and 1 year in order to maximize average PoS difficulty and to minimize deviation about that mean.
Current client behavior includes both splitting and combining functions to achieve some equilibrium size window for minting outputs.
This proposal seeks to give the user more direct control over these client functions to support the various needs of users with varying amounts and time frames.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- "SPLIT": When a coinstake is used to mint a block within a short period of time from the creation of the staking output, the client will output the coinstake transaction to two approximately equal outputs.
- "COMBINE": The addition of more than just the minting input to the coinstake transaction.
- "MINTING OUTPUT" or "STAKING INPUT": The input to the coinstake transaction that is hashed and compared against the PoS difficulty to find a block.

## Motivation

When an large minting output finds a block in a short period of time after its creation, it splits into two outputs that share the block reward.
This is done because of the 30 day period required for an output to wait before it mints, such that each output can only find a single block at most once every 30 days.
By splitting outputs, the client increase the difficulty and stability of the network, allowing a single large user to find multiple blocks a day.

A user with many outputs may be attempting to mint outputs that are very small and will not mint for a very long time.
In some cases, the client will combine these small outputs into a single coinstake transaction.
In these instances, the minting output is used as a vehicle achieve a more efficient output distribution for minting without sacrificing money, time, or coinage.

The maximal strategy for minting is bounded on either side by network effects and given incentive through the dynamic/static block reward introduced in RFC-0018 and Peercoin v0.9.
On the side of splitting, we have the 90 day period required for an output to reach full maturation and maximal mint probability with a hard limit at the 30 day minimum for minting.
On the side of combining, we have the one-year limit on rewarded coinage such that outputs with a longer expected time to mint will not attain their full reward due.

This proposal will model 3 types of users: a) those that do not want the client to affect the output size aside from increasing it during minting, b) those that want the client to seek out optimal behavior, and c) those that wish to determine the exact behavior themselves.

## Detailed design

A basic toggle to turn off splitting/recombine can be addressed in the client first, but we will not focus the discussion of this RFC there.
However, it is worth mentioning up front that the user-interface is really the point of this proposal.
How these options are presented allows users potentially fine control over client behavior.

These additional choices can be placed in the Settings>Options>Wallet tab of the client, and will be represented by 2 integers and a boolean.
The first is the threshold below which outputs will split, labeled 'Split Intensity', which is a number between 30 days and one year (or potentially infinity).
The second is the target percent chance to mint after a year, labeled 'Combine Threshold', which is a percentage between 0% and 99.999% (or potentially 100%).
The boolean will grey out the other two inputs and set the Split Intensity to 30 and the Combine Threshold to 0%.

Outputs minted in less time than the Split Intensity will split into two.
The current effective value for this parameter is 90 days, corresponding with the maturation period.
This 90 day value can be taken as default, and the 30 days required minimum sets a deterministic lower limit.
The upper limit could be set very large, a situation that we will discuss in the #Drawbacks section.
As a practical approach, we will set the limit at 1 year corresponding with the coinage limit.

Small outputs will merge with the coinstake of other minted outputs when the amount is below a certain threshold.
Currently that amount is 1/3 of the PoW block reward, the relevance of which is waning.
This proposal would instead add a 1-year probability calculation to the mint tab, then use this mint probability to determine which outputs to combine.
The threshold would be set at 50% by default, resulting in a client behavior that seeks to combine outputs that have a <50% chance to mint in the next year.
The minimum of 0% is a natural way to turn off recombine, but the upper limit has a similar ambiguity to the Split Intensity and we will discuss the unbounded case in the #Drawbacks section.
A limit of five 9's assurity is a statistically relevant and rational upper limit.

## Drawbacks

*Unbounded Split/Recombine*  
Minters may seek out network conditions that are favorable for their own reward and detrimental to the overall network difficulty.
The static and dynamic portions of the reward provide buffers on both ends of this spectrum, and result in an equilibrium that is close to the ideal conditions.
As such, if the user achieves a maximal reward state via some delicate tuning of the paramaters introduced in this rfc, it may not be the exact ideal configuration for maximal network difficulty, but it will be close.

As a particular example, we should look to the case of maximal and unbounded splitting and recombining.
In this case, the client tries to combine every output they are allowed to, and splitting as often as possible.
We want to ask the question: is this an abuse of the statistical rules to get a leg up, or is it just a user setting bad parameters whose function should be limited?
To address this, we imagine a single large output with an average mint time of 60 days, well within the maturation window but not immediately after the minimum required 30 days.
If the users splits this output into 5 when it mints, then recombines them all the next time any of those 5 mint, rinse and repeat, does the user gain any benefit?
Of course the standard client would only split into two, but there is nothing that fundamentally prevents this more extreme version, and it is instructive to think about this aggressive form.
Minting in Peercoin involves hashing each output each second and comparing against a target set by the coinage and the network difficulty.
The statistical representation of splitting is like comparing the sum of multiple rolls of smaller dice to the roll of a single large die: the expectation value is the same, but the variance is lower.
As such, the user does not gain any additional expectation of reward, but does potentially gain a more even distribution of that reward throughout time.
If this thought experiment holds true, what prevents all users from turning their splitting and recombine to their most aggressive values to have a more regular statistical distribution of mints?  Is that behavior even undesired, given that more regular minting means lower variance of block timing?

## Alternatives

*Recombine based off Static Subsidy*  
In giving the user access to the recombine function, it would be more intuitive and have wider application if it was simply listed in units of 'PPC' such that outputs with an amount less than the specified number will be recombined.
In this case, the default value becomes somewhat more controversial, but a good rule of thumb would be something like 10 times the static subsidy.  At the time of writing this would be ~12.6 PPC and would nearly coinincide with the 1/3 PoW block reward combine limit that is currently being applied.

*Protocol Rules for Recombining*  
Combining is currently done when an output is smaller than 1/3 of the PoW block reward.
Any output that is combined in the coinstake will contribute to the total PoS reward.
This is a client behavior, and there are no protocol rules preventing this threshold from being increased to an arbitrary size to maximize interest.
By imposing a maximum recombination size of 10 times the static subsidy and similarly changing the client behavior, we can restrict behaviors that attempt to exploit the recombine mechanics.
In this case, the user-controlled parameter would only have a range between 0 and this limit.

## Unresolved questions

*Coinstake Pooling*  
The 'combine' client behavior includes a restriction that all inputs be from the same address.
This is done to maximize client privacy when minting by preventing addresses from being identified as part of the same client.
If we created a 'pooling' option in the client that relaxes this rule then small minting outputs from less wealthy addresses could make use of the presence of richer addresses in order to mint in reasonable time frames.
In conjunction with RFC-0012 where these mint inputs might be signed by mint keys and are not necessarily owned by the same users, this could be a powerful feature for mint pools.

The behavior should be such that the outputs to the coinstake should be respective to the addresses, amounts, and rewards of each input, as well as the potential for the minting coinstake to split.
This way, the default client can easily be used to create minting pools using mint keys from several independent holders of the spend keys.
By promoting simple in-client behavior, we can promote a large number of mint pools to alleviate the centralization issue that is the main drawback of RFC-0012.
