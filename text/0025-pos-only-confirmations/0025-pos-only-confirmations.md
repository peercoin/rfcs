# PoS Only Confirmations

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
PoW blocks on the tip of the chain will not be considered when counting confirmations in the core client implementation.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

PoW blocks carry minimal chainweight and any number of PoW confirmations can be overwritten by a single PoS block.
Requiring at least one PoS block of confirmation for PoW blocks provides more security and stability than if dangling PoW blocks are included in the count.

## Detailed design

When calculating number of confirmations for a transaction, the most recent PoS block (called the 'last PoS block') will be used in place of the current block height.
Core protocol such as adding blocks and verifying blocks will not be affected.

RPCs:
getblockcount will be unaffected.
getreceivedby{} will only count confirmations from the last PoS block.
gettransaction will return the confirmations from the last PoS block.
listreceivedby{} will only count confirmations from the last PoS block.
listsinceblock will start from the last PoS block for target-confirmations.
sendfrom will require confirmations from the last PoS block.

## Drawbacks

Confirmation of a transaction will take longer on average.
Calculating the exact effect is complex, but the target spacing ratio of 1:6 PoW:PoS allows us to approximate a number around 17%.

Basing 'getblockcount' off of a different height than the other RPCs could hypothetically cause issues in existing third party code.

## Alternatives

## Unresolved questions?
