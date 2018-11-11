# Round Transaction Fees Up To 0.001

- Status: implemented
- Type: enhancement
- Related components: `protocol`
- Start Date: 16-May-2018
- Discussion: https://github.com/peercoin/rfcs/issues/8
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: backpacker69

## Summary

Currently transaction fees are rounded up to nearest 1 hundredth of PPC and any small transaction under 1024 bytes must pay at least 0.01 PPC to be included in the block. Transaction of 1025 bytes must pay at least 0.02 PPC to be considered, etc.

It is proposed to have fee rounding changed in such way that all transactions will pay exact proportional to size fee for the right to be included in the block, at unchanged rate of 0.01 PPC per kilobyte, with 0.001 PPC minimum.

Effectively, transactions up to 102 bytes will pay minimal fee of 0.001 PPC. Transactions of bigger size will pay unrounded fee proportional to their byte size.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

We want to reduce transaction fees to ease adoption of our blockchain. 

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody familiar
with the network to understand, and for somebody familiar with the code practices to implement.
This should get into specifics and corner-cases, and include examples of how the feature is used.

## Drawbacks

Fewer peercoins will be burned as fees.

## Alternatives

We could reduce cost per kb. 

## Unresolved questions

What parts of the design are still to be done?
