# Coinage Limit

- Status: agreed
- Type: protocol adjustment
- Related components: 
- Start Date: 04-27-2020
- Discussion: 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: Nagalim

## Summary

A Peercoin holder that waits years to mint an output is barely participating in the network and should not be rewarded for all the time spent holding without minting.
On the other hand, we want the to keep at a minimum the number of coins required in order to participate in minting and be fully rewarded.
As such, we will impose a 1 year limit on the awarded coinage of an output.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- A '1-year' period is considered to be (365 * 33 + 8)/33 days
- Periodic minting is considered to be when a minter disconnects from the network for long periods of time between minting.
- Continuous minting is when a minter does their best to keep their client on the network at all times, rarely missing an opportunity to form a block.

## Motivation

Limiting coinage disincentivizes extremely long term periodic minting, thereby making more continuous minting more attractive.
On the other hand, the size of the limit can cause small outputs to take a long time to mint even when minting continuously.
As such, a balance should be struck that is considered a reasonable amount of time.
One year is chosen in the context of a 90 day ramp up to maximum mint probability.

## Detailed design

When calculating rewards for finding a PoS block, limit the coinage rewarded to 1 year.

## Drawbacks

Small outputs may not have a chance to mint within 1 year.
The downside of this can be seen as 2-fold.
Firstly, very small coin holders will lose some reward even when continuously minting.
Secondly, big coin holders will not want their outputs to be split too small.
As output splitting of continuous minters can cause the PoS difficulty to increase, this behavior should not be discouraged.
Still, if the output does not have a majority chance of minting within a year, then the benefit of splitting further is small (On the order of 8%, assuming constant difficulty).

To calculate this percentage change on the difficulty, we first look at the time it takes an output to reach maturity.
While it is 90 days, the probability of minting the output is increasing from the 30 day to the 90 day mark.
We can therefore assume that the amount of mint-time-at-full-maturity lost during this maturation period is 60 days.
We then compare two cases: case A has 1 output with an average mint time of 1 year, and case B has 2 outputs with average mint times of 2 years each.
While case B will likely mint twice at the 2 year mark, case A will mint, then has a maturation period before minting again.
Case A will therefore likely mint its second block at the (2 years + 60 days) mark, a difference of about 8%.

## Alternatives

While decreasing the limit would exacerbate the drawbacks, increasing the limit to something like 2 years would mitigate the drawbacks, though it would also reward periodic minters that wait over a year between mints.

## Unresolved questions

This proposal was split off from RFC-0011.
Is it still beneficial without the requirements introduced in RFC-0011?
