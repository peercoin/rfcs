# Increase OP_RETURN size limit

- Status: proposed 
- Type: enhancement
- Related components: protocol
- Start Date: 16-05-2018
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this) 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: backpacker69

## Summary

Increase OP_RETURN size limit to 1024 bytes

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

We want to increase adoption of our blockchain, allowing more data to be stored on the blockchain without polluting utxo table is good opportunity.

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody familiar
with the network to understand, and for somebody familiar with the code practices to implement.
This should get into specifics and corner-cases, and include examples of how the feature is used.

## Drawbacks

Blockchain size will increase

## Alternatives

We could remove OP_RETURN size altogether as long as appropriate fee is paid

## Unresolved questions

What parts of the design are still to be done?
