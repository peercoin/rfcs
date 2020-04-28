# Proof-of-Stake Reward

- Status: proposed
- Type: protocol adjustment
- Related components: 
- Start Date: 04-28-2020
- Discussion: 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: RFC-0011
- Author: Nagalim

## Summary

While PoS inflation can currently be up to 1% per year, in reality only a fraction of the existing coins participate in the minting process.
This RFC will increase the reward of all participants, while also introducing a supply-based component to the reward that will motivate more continuous minting.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- An 'Attack' is generically any action performed in an attempt to gain an unfair advantage when using the Peercoin protocol.
- A 'Nothing at Stake Attack' (N@S for short) is defined as any minter that intentionally mints a block on a chain that is not the longest valid chain.
- A 'Stake Grind Attack' is defined as any minter that intentionally splits his/her inputs in order to alter the odds of minting a block.
- A '1-year' period is considered to be (365 * 33 + 8)/33 days

## Motivation

Minter participation is not at a very high level given the current supply of Peercoin, which is a detriment to chain security.
Increasing PoS block reward can be used as a strategy to increase minter participation.
In addition, a static rewards have been used by other blockchains to promote continuous participation in the block creation process.
Tieing the static component to the total coin supply can be used as a method of ensuring continuing relevance of that component.

## Detailed design

The PoS reward will be increased to 2% annually.
An additional quantity will be added to this that is based off the total money supply.
As such, an index tracking money supply will be included in the client.
Take the target number of PoS blocks in a year to be:
> YearlyBlocks = [(365 * 33 + 8) / 33] * 1440 / nBlockTarget

Then the total reward should be:
> Subsidy = (0.02 * coindays) + 0.002 * MoneySupply / YearlyBlocks

## Drawbacks

*Increased Complexity due to Money Supply Index*  
Tracking the money supply will need to be done by all clients seeking to verify the coinstake.
This will need to be maintained as an additional index into the future of the Peercoin code.
The impact on system requirements of the node will likely be negligible.

*Increased PoS Inflation*  
Currently, the PoS inflation can be anywhere from 0% to 1%.
This protocol change will increase that range to 0.2% to 2%.
Historically, no more than 50% of the network should be expected to participate in the minting process, so this modification will likely result in under 1% inflation.

*Stimulating Splitting*
The Peercoin client has always had a splitting function where it splits in half outputs that mint before the 90 day maturation period.
A component of the reward that is not proportional to coindays will create competition between minters and benefits users that reduce their output size.
While this is desirable for chain security, any component of this nature will motivate users to split down to dust values (i.e. a stake grind attack), which greatly bloats the UTXO table and creates a barrier of entry and exit for competitive minting.
For this reason, a coinage limit like that detailed in RFC-0017 should be used to impose a minimum output size to receive the full benefits from the coinage component.

## Alternatives

The numbers chosen in implementation of this proposal are integral to the effect it will have on the network.
The proposal as written above can be called '2%+0.2' to represent the coindays component as 2% and the supply-based component as 0.2%.
This ratio of 1:10 can be taken approximately as the participation level where stake grinding to dust values becomes profitable.
It is also related to the pressure a minter feels to participate continuously as opposed to periodically.
A ratio between 0.1 and 0.2 should be deemed as appropriate to stimulate minting participation above 10% or 20%, without creating a situation where stake griding is the norm.

As such, other proposals such as '3%+0.25' or even '10%+1' can be considered.
At current network participation and PoW miner reward, even the '10%+1' option will not cause PoS to dominate the inflation caused by PoW.
The '3%+0.25' option is more closely akin to a 1% inflation rate at current network participation, though it should be noted that participation is historically low at the time of writing.

## Unresolved questions?

Upgrading the protocol from RFC-0018 to RFC-0011 would not cause huge issues.
However moving from RFC-0011 to RFC-0018 would require long-term upkeep of the adjustment variable, and would not be an efficient design path.
