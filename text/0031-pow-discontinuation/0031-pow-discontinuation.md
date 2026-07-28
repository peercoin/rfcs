# PoW Discontinuation

- Status: proposed
- Type: protocol adjustment
- Start Date: 20-07-2026
- Author: MatthewLM

## Summary

This RFC proposes to discontinue Proof-of-Work, allowing Peercoin to become an
efficient pure Proof-of-Stake coin.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

PoW has been used to distribute coins since inception and has ensured the fair
distribution of Peercoin for almost 14 years with no premine. When Peercoin was
launched in 2012, GPU mining was still dominant. Now mining is the preserve of
industrialised ASIC mining farms, outside the reach of ordinary people.

PoW is not used to secure the blockchain. Only PoS is.

PoW has done its job of distribution with a long opportunity to mine Peercoin.
The ongoing use of PoW only serves to dilute existing holders with a cost that
is no longer necessary.

Continued PoW issuance creates unnecessary downward pressure on the price.
Resources are used with negative environmental impact.

Moving to pure PoS makes Peercoin easier to understand with a clear narrative
of efficiency.

## Detailed design

The client shall reject PoW blocks after the activation of a fork using
`IsProtocolV16`. Rejection of PoW blocks shall be done when checking a block or
block header before the work is checked. `CheckBlockHeader` may be a good place
to do this check.

The threshold of the fork shall be 75% and shall not activate until the 19th of
August, 2017 which is the 15th anniversary of Peercoin.

## Drawbacks

The removal of PoW may cause some miners to lose awareness of Peercoin. Existing
miners may feel left out.

## Alternatives

PoW could be discontinued after the Peercoin supply reaches a certain level, but
this is difficult to lock-in reliably due to the possibility of the supply
falling below that level afterwards.

PoW could be ended gradually by reducing the reward by a factor over time until
the reward reaches zero, or the difficulty could be gradually increased causing
PoW blocks to become increasingly uncommon.
