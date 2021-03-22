# Proof-of-Work SHA3 for Energy Efficiency

- Status: proposed
- Type: adaptation
- Related components: 
- Start Date: 03-21-2021
- Discussion: https://talk.peercoin.net/t/rfc-0021-proof-of-work-sha3-for-energy-efficiency/15283
- Author: Paden

## Summary

Peercoin PoW blocks could be discovered with low power silicon, such as the Apple M1, by migrating to SHA3 thus ensuring that block rewards would have a more fair distribution of new coins on modern, energy efficient devices.

## Motivation

Peercoin is uniquely positioned as the leading energy efficient crypto ready for mobile deployment.
RFC0019 has introduced an orphan block rate of near zero.
The blockchain has experienced the smallest rate of growth per year, reaching only 1.2GB in 9 years.
Mining of cryptocurrency has become a worldwide problem. Energy usage wasted in block rewards competitions is causing irreversible environmental damage. 

## Detailed design

Decreasing block rewards for PoW offers incentive to spread risk and provide more reward to individual low-power miners in one common PPS pool where an algorithm will maximize reward output for the least amount of consumed power across all devices in the pool.
Staking and mining would both be possible on mobile by developing around the ARMv8 architecture for iOS, MacOS, Android, Linux, and BSD; discouraging the need for any other application-specific hardware which sequesters limited precious resources into economic weapons of mass destruction.

## Drawbacks

Changing the PoW algorithm to SHA3 will render investments in SHA256 miners obsolete on the Peercoin network in favor of distributed low-power mining for energy efficiency.

## Alternatives

## Unresolved questions?

