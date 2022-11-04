PoS Only Confirmations

- Status: proposed
- Type: protocol adjustment
- Related components: 
- Start Date: 11-04-2022
- Discussion: 
- Supercedes: (fill me in with a link to RFC this supersedes - if applicable)
- Requisite: 
- Author: Nagalim

## Summary

PoS blocks carry the vast majority of chainweight and are the foundation of blockchain security in Peercoin.
Only PoS blocks will be considered when counting confirmations in the core client implementation.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

PoW blocks carry minimal chainweight and any number of PoW confirmations can be overwritten by a single PoS block.
Using PoS blocks alone for confirmations provides more security and stability than counting PoW and PoS blocks together.

## Detailed design

The standard client implementation will only consider PoS blocks when counting confirmations for a transaction.
There will be no change to the core protocol, only the client itself.

## Drawbacks

Confirmation of a transaction will take ~17% longer on average, given a 6:1 PoS:PoW block ratio.

## Alternatives

## Unresolved questions?
