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

The 'sweet spot' for minting an output is between 90 days and 360 days in order to maximize average PoS difficulty and to minimize deviation about that mean.
Current client behavior includes both splitting and combining functions to achieve some equilibrium size window for minting outputs.
This proposal will refine that window by reducing the split frequency to 75 days, changing the combine behavior, and imposing additional protocol restrictions on the combining process by limiting the combination amount to 1/6 of the staking input.
This proposal will also outline a possible method for creating 'pooled coinstakes' in combination with RFC-0012 using an option in the standard client.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- "SPLIT": When a coinstake is used to mint a block within a short period of time from the creation of the staking output, the client will output the coinstake transaction to two approximately equal outputs.
- "COMBINE": The addition of more than just the minting input to the coinstake transaction.
- "STAKING INPUT": The input to the coinstake transaction that is hashed and compared against the PoS difficulty to find a block.

## Motivation

Splitting is currently done whenever an output that is less than 90 days old is used to mint a block.
With this aggressive split frequency, and with a changing PoS difficulty, minters can easily end up with outputs that do not mint for over a year.
We can see the likelyhood of this because the chance to mint does not change after 90 days at constant difficulty, so if an output is minted just before the 90 day interval, then we can expect its split outputs might not mint until 180 days (because the outputs are half the size).
Over a long period of time, random chance could split one of these outputs again, doubling this length to 360 days, at which point there is reasonable expectation for some outputs to take longer than a year to mint.
Reducing the split frequency to 75 days would require random chance (~2 splits beyond the likelyhood) or a long period of time to cause the same effect.

Combining is currently done when an output is smaller than 1/3 of the PoW block reward.
Any output that is combined in the coinstake will contribute to the total PoS reward.
This is a client behavior, and there are no protocol rules preventing this threshold from being increased to an arbitrary size to maximize compounding interest.
By imposing a maximum combination size of 1/6 the size of the staking input and similarly changing the client behavior, an output that has been split 3 times more than another output (1/8 output size) will be combined and reduce the chance that an outlier will not mint for longer than a year.
This protocol rule will also prevent users from taking advantage of the combine process to mint all their outputs every time they find a block, which would reduce overall PoS difficulty.

## Detailed design

Splitting will be done when a minting output is <75 days old, rather than 90 days.

A protocol rule will be added that the sum of all inputs to the coinstake other than the staking input must have a total amount that is less than (or equal to) 1/6 of the amount of the minting input.

nCombineThreshold will be changed to 1/6 of the staking input.

## Drawbacks

Splitting is used to increase PoS difficulty and mint regularity.
By changing the default client behavior to reduce the proliferation of small outputs for minters, we will reduce the overall PoS difficulty.
The choice of 75 days for the split frequency should be considered a small, conservative adjustment that will have only a minimal impact on PoS difficulty.

Changing the combine rules could similarly affect PoS difficulty.
Also, addition of a restriction on inputs to the coinstake could affect applications related to pooling together minting coins.
The concept of 'coinstake pooling' will be addressed in the 'Unresolved questions' section.

## Alternatives

Choosing 60 days instead of 75 days would be a more aggressive version of this proposal.

This proposal has 3 parts: changing the split frequency, addition of a protocol rule, and changing the combine behavior.  Any of these three parts could be implemented independent of the other parts.

## Unresolved questions

*Coinstake Pooling*  
The 'combine' client behavior includes a restriction that all inputs be from the same address.
This is done to maximize client privacy when minting by preventing addresses from being identified as part of the same client.
If we created a 'pooling' option in the client that relaxes this rule then small minting outputs from less wealthy addresses could make use of the presence of richer addresses in order to mint in reasonable time frames.
In conjunction with RFC-0012 where these mint inputs might be signed by mint keys and are not necessarily owned by the same users, this could be a powerful feature for mint pools.

The behavior should be such that the outputs to the coinstake should be respective to the addresses, amounts, and rewards of each input, as well as the potential for the minting coinstake to split.
This way, the default client can easily be used to create minting pools using mint keys from several independent holders of the spend keys.
By promoting simple in-client behavior, we can promote a large number of mint pools to alleviate the centralization issue that is the main drawback of RFC-0012.

It should be noted that acceptance of the static subsidy introduced in RFC-0011 as well as the transaction fee for coinstakes larger than 1KB may create ambiguity surrounding which outputs are owed what portion of the subsidy.
