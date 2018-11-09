# Increase OP_RETURN size limit

- Status: agreed 
- Type: enhancement
- Related components: protocol
- Start Date: 16-05-2018
- Discussion: https://github.com/peercoin/rfcs/issues/10 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: backpacker69

## Summary

Increase OP_RETURN size limit to 256 bytes.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

We want to increase adoption of our blockchain, allowing more data to be stored on the blockchain without polluting utxo table is good opportunity.

## Detailed design

A simple parameter change.

## Drawbacks

Blockchain size may increase.

## Alternatives

We could remove OP_RETURN size altogether as long as appropriate fee is paid.

## Unresolved questions

What parts of the design are still to be done?
