# Proof-of-Work reward cap

- Status: proposed
- Type: protocol adjustment
- Related components: 
- Start Date: 03-18-2021
- Discussion: https://talk.peercoin.net/t/rfc-0022-pow-reward-cap/15282
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by:
- Author: Peerchemist

## Summary

Peercoin PoW block reward is product of network Proof-of-Work difficulty: the higher the network difficulty, the lower the block reward. This mechanism is effectively "pegging" the issuance of Peercoins to the development of SHA256 ASIC miner hardware. Reasoning behind this decision is that it's expected that efficiency of SHA256 miners represents the level of advancement of general cryptocurrency industry, at least that is something that can be encoded into an algorithm. Miners will turn to Peercoin when profit is high, and drive up the PoW difficulty while reducing the block reward. Which is expected behavior, to have ever decreasing average block reward.
However, PoW miners have no loyalty, they will move on to mine whatever is most profitable at the moment. Sometime they will even speculate about future price appreciation of a specific asset and will dedicate time and energy to mine it at loss for a while. This can lead to extended periods of lower mining difficulty and relatively large PoW block rewards. It is easy to imagine some extremely unusual chain of event (black swan) which would result in extremely low PoW difficulty and thus abnormal PoW-based inflation.

It is justifiable to cap the Pow block reward at 50 Peercoin in order to shield the network from excessive inflation in case of unusually low PoW difficulty. 

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Peercoin uses PoW block rewards as fair method distribution method for new coins. Peercoin is using same algorithm as Bitcoin and Bitcoin Cash and is thus part of the same sha256 mining ecosystem - which in effect means that conditions and economics of Peercoin mining change to reflect changes in market dynamics of Bitcoin and Bitcoin Cash mining. Peercoin PoW block reward is product of network Proof-of-Work difficulty: the higher the network difficulty, the lower the block reward. This mechanism is effectively "pegging" the issuance of Peercoins to the development of SHA256 ASIC miner hardware. Reasoning behind this decision is that it's expected that efficiency of SHA256 miners represents the level of advancement of general cryptocurrency industry, at least that is something that can be encoded into an algorithm. Due to ever-growing total sha256 hash-rate, Peercoin block reward averages under 50 Peercoin / PoW block. Sometime market dynamics reflect negatively and Peercoin PoW difficultly plummets which results in higher than usual block reward.
Miners will turn to Peercoin when profit is high, and drive up the PoW difficulty while reducing the block reward.
However, PoW miners have no loyalty, they will move on to mine whatever is most profitable at the moment. Sometime they will even speculate about future price appreciation of a specific asset and will dedicate time and energy to mine it at loss for a while. This can lead to extended periods of lower mining difficulty and relatively large PoW block rewards. It is easy to imagine some extremely unusual chain of event (black swan) which would result in extremely low PoW difficulty and thus abnormal PoW-based inflation.

Peercoin is mined and distributed since it's inception. Overwhelming majority of all Peercoins ever created come from PoW block rewards. Even today, over 8 years since the network genesis, less than 1M of Peercoins have been created through PoS rewards. While remaining 25.8M are result of PoW block rewards. [1](https://peercoinexplorer.net/charts/annualinflation/5/linear/default)

It is justifiable to cap the Pow block reward at 50 Peercoin in order to shield the network from excessive inflation in case of unusually low PoW difficulty. 


## Detailed design

Set protocol rule which defines 50 Peercoin as maximum PoW block reward. Reject blocks with higher reward.

## Drawbacks

It is possible that this could cause PoW reward to be inadequate sometime and cause miners to drop out.

## Alternatives

Modify the PoW reward algorithm to handle this problem differently.

## Unresolved questions?

