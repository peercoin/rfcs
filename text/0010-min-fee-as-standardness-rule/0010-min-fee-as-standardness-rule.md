# MIN_FEE as standardness rule

- Status: rejected
- Type: enhancement
- Related components: protocol
- Start Date: 16-05-2018
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this) 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: peerchemist

## Summary

Make MIN_FEE variable a standardness rule, which defines whether transactions gets relayed, and not a hardcoded protocol rule.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Peercoin economic model is set around hardcoded transaction fee which is transaction size in kB * 0.01 Peercoin / kB, however it starts from 0.01 Peercoin which is lowest possible fee. This rule (MIN_FEE) is protocol level rule and changing it requires network to execute hard fork and set new set of rules.
Making this rule a standardness rule which defines whether transaction gets relayed or not allows for easier tweaking of the rule and removes the need for network hardforks.

## Detailed design

Make future changes to MIN_FEE variable simpler without requiring network hard fork.

## Drawbacks

Hard fork is required to implement this.

## Alternatives

Modifying the MIN_FEE in src/main.cpp every time there is consensus that it needs to be change and hard-forking the network each time.

## Unresolved questions

What parts of the design are still to be done?
