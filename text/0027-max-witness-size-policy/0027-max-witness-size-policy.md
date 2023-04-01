# Maximum Witness Size Policy

- Status: proposed
- Type: policy change
- Start Date: 31-03-2023
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this) 
- Author: MatthewLM

## Summary

Given concern over Bitcoin "ordinal inscriptions", this RFC proposes restricting
the size of each input's witness data for standard transactions to 100KB. Nodes
that use the new policy will not relay, mint or mine transactions with large
inscriptions.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

The ordinal protocol may be ported to Peercoin which could cause large amounts
of superfluous, non-transactional data to be included in blocks. Other protocols
may also utilise Taproot witness elements to add large amounts of data.

Limiting the witness data size for each input through a policy change mitigates
against protocols that use excessive amounts of data within transactions. In
particular, this change deters large inscriptions of the ordinal protocol which
is a particular concern.

## Detailed design

The standard transaction policy of the Peercoin client shall be changed. It is
recommended that alternative software abide by the same policy unless there is a
specific reason to deviate from the policy.

A transaction shall be non-standard if any input has total witness data that
exceeds 100KB. To simplify calculation of the total witness data for each input,
it shall be calculated by summing the size of each witness element in the stack
without including varints.

The `IsWitnessStandard` function in `src/policy/policy.cpp` shall return false
under the new non-standard condition. The rule shall apply regardless of the
witness program version. Any changes shall only apply to standard transaction
policy and must not affect consensus.

## Drawbacks

It remains possible for transactions with large witness data over 100KB per
input to be included in blocks by non-standard nodes. It is also possible to
include 100KB of data in multiple inputs such that the total transaction witness
data is much larger.

Restricting the size of data in transaction inputs restricts the possible use
cases of Peercoin. This includes ordinal inscriptions but may impact other
protocols that rely on large amounts of input data.

## Alternatives

A consensus change may limit witness data or provide a fee-based mitigation but
this requires a new fork which would interfere with the ongoing v0.12 fork. A
consensus change is less straight-forward and imposes stricter limits that are
not as simple to change.

100KB is a compromise. A larger limit will allow for larger ordinal inscriptions
and other possible data uses but encourages more superfluous data usage on the
chain. A much smaller limit of ~5KB should cover multisig inputs of 40 keys or
more, but places a heavier limit on applications that depend upon data
availability, including ordinal inscriptions.

