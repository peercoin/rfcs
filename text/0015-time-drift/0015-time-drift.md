# Time Drift

- Status: proposed
- Type: parameter modification
- Related components: (if any)
- Start Date: 15-February-2020
- Discussion: 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: Nagalim

## Summary

The current time drift allows blocks to be included in the chain that are at most 2 hours beyond a nodes current time.
Standard internet-connected machines use Network Time Protocol these days and are generally synchronized with other clocks around the world to a fine degree.
Tightening up the time drift will provide more fidelity in blockchain time with little drawback.
For these reasons, the proposal is to reduce the drift time to 15 minutes.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

A tighter blockchain time reduces time-drift abuses like minting an hour into the future, and allows for additional features or modifications that rely on a time-drift that is on the order of the target PoS block time (10 minutes).

## Detailed design

nMaxClockDrift = 60 * 15; //15 minutes

## Drawbacks

The drift-time should be long enough for blocks to propagate around the world, which can sometimes take longer when moving through various firewalls.
If the time is not long enough, then blocks will be orphaned on some nodes and not others, causing forks.
These forks will likely be resolved within a block or two, but frequent orphans are not desirable for a stable chain.
Care should be taken not to lower the drift-time below the time it takes a block to travel between the most remote locations on the network.

## Alternatives

Other drifts could be considered, such as 20 minutes or 10 minutes.

## Unresolved questions

