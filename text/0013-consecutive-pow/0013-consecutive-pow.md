# Consecutive PoW

- Status: rejected
- Type: new feature
- Related components: (if any)
- Start Date: 25-January-2020
- Discussion: https://talk.peercoin.net/t/rfc-0013-consecutive-pow/10049
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: Nagalim

## Summary

Banning consecutive PoW blocks will empower PoS security as the dominant security method for Peercoin.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

The Peercoin network uses the same hashing algorithm as PoW dominant chains such as Bitcoin, but its security is known to be PoS-dominant.
PoW in Peercoin is used only as a method of distribution, so a consecutive stream of PoW blocks threatens that paradigm.
In the current state of the cryptocurrency ecosystem, it is easiest for large miners to avoid the Peercoin network until the hash power vrs price is low enough to make a profit, then move their hash power onto the network and spike the PoW difficulty with several blocks in succession.
This has resulted in a large number of consecutive PoW blocks from a single source that threaten the security of the chain, as well as large reorgs of 6 or more blocks as PoS blocks replace the PoW string with higher chain weight.
In addition to the threat of a double spend, reorgs in general are detrimental to chain stability.
Second layer solutions and use of the blockchain as an immutable settlement layer can also be compromised if the block depth of a transaction is not a clear and consistent truth.

In addition to stability concerns, there is the concern of an 'instamine', or the rapid output of a large number of Peercoins when a large miner hops onto the network.
This is a concern because the miner may immediately drop off the network again, resulting in a large reward for causing instabilities in the chain, which is far from ideal.
We should seek to limit this behavior, in order to more evenly spread the rewards amongst more continuous and reliable miners.

## Detailed design

The Peercoin network will ban consecutive PoW blocks, such that any PoW blocks built ontop of other PoW blocks will be orphaned.
The result will be that, going forward, each PoW block must be preceded and followed by a PoS block.

## Drawbacks

*Miner Participation*  
Otherwise valid PoW blocks will be rejected.
This may reduce miner participation, which is a serious concern given that lack of a continuous miner participation is the primary reason this RFC is being proposed in the first place.
On the other hand, it could be argued that this RFC will prevent miners from having large strings of PoW blocks orphaned unexpectedly by a single higher chainweight PoS block.
This RFC will mostly affect large changes in PoW difficulty, and under conditions of constant difficulty will only cause a small stochastic increase in PoW orphan frequency.

*PoW Block Usurping*  
With or without the changes introduced in this RFC, a minter with a large hash power can use their coinstake to secure a PoW block that replaces a PoW block on the end of the chain, even if the minter's PoW block was found at a later time (lower difficulty).
However, without this RFC, that minter could receive just as much reward by placing the PoW block after the current one.
With this RFC, if the chain ends in a PoW block the minter must use their hash power on another chain until a PoS block is found, or attempt to usurp the current PoW block by using their stake to secure it.
While this does not compromise the chain beyond a single orphan, an attack of this nature would result in a lowering of the PoW difficulty directly and cause miners to avoid the Peercoin chain.

*Difficulty Disequilibrium*  
Because PoW blocks could no longer come out in rapid succession when the hashpower changes, the difficulty will take longer to achieve equilibrium.
In the case of a momentary large hashpower change, this will prevent large spikes in the difficulty.
However, in the case of a more long term hashpower change, this will cause the system to enter a state of difficulty disequilibrium, where PoW blocks are found quickly after each PoS block is found.
In this state, the miner who receives the PoS block first has an advantage over other miners.
Because the Peercoin chain recalibrates difficulty every block and the minter network is well dispersed globally, this drawback is temporary (on the order of hours or less) and not easily gamed.

## Alternatives

*PoS-Only Confirmation*  
Only counting PoS blocks when calculating the 6-block transaction validity will also prevent double-spends in the event of a large miner moving to the Peercoin network, even if a string of consecutive PoW blocks are found.
This method will not affect miners negatively as the current proposal would, but it could result in longer and less-intuitive wait times for the right type of confirmations.
In addition, while it resolves the danger of a double spend, it does not stabilize the chain beyond that.
A large string of PoW blocks with tiny chainweight can still be orphaned by a single PoS block, causing instability in the chain that can affect second layer solutions.

*Difficulty Spiking*  
By spiking the PoW difficulty in the event of consecutive PoW blocks, a more organic solution to this issue could be found.
For example, doubling the PoW difficulty every block until a PoS block is found (at which point the multiplier is reset) would require 36x the hashing power to find the 6th block in a row compared with the first block.
In addition, the PoW reward is halved every time the difficulty increases by 16x, so every 5th consecutive block would be half again as rewarding.
This is not as abrupt a solution as the proposed RFC, but is effective assuming that the change in hashpower is not many orders of magnitude.
This solution could also result in chains of 2 or 3 consecutive PoW blocks quite easily, but could be considered more acceptable to miners than banning consecutive blocks entirely.
It is worth noting that any arbitrary function could be chosen for the increase of PoW difficulty, and that if the difficulty were to be multiplied infinitely after the first block we would recover the proposed RFC.

## Unresolved questions

What will miners do when a PoW block ends the chain?  Will they use their hashpower on a different coin?  Will the mining software support such an abrupt switchover and switch back when a PoS block is found?
