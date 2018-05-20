# Title-cased Name of the Feature

- Status: proposed
- Type: enhancement
- Related components: protocol
- Start Date: 05-16-2018
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this) 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: saeveritt

## Summary

Remove the 520 push byte size limitation imposed on MAX_SCRIPT_ELEMENT_SIZE. 
No imposed hard limit.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

The 520 byte size limit restricts Pay-to-Script-Hash as the entire script must respect the push limit. This constrains Pay-to-Script-Hash multisignature scripts to n-of-15. Any larger and it becomes unspendable.

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody familiar
with the network to understand, and for somebody familiar with the code practices to implement.
This should get into specifics and corner-cases, and include examples of how the feature is used.

## Drawbacks

Requires a hard fork.
In the future this could result in unwanted attacks for cross-chain atomic swaps (if current standard swap contracts are used).  
If we remove the 520 byte limit then we must standardize atomic swap contracts to the proposed version here:
https://gist.github.com/markblundeberg/7a932c98179de2190049f5823907c016

## Alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions

What parts of the design are still to be done?
