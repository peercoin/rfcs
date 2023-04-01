# Max Tx Size Consensus Limit

- Status: proposed
- Type: protocol consensus rule
- Related components: `consensus`
- Start Date: 28-Mar-2023
- Discussion: https://github.com/peercoin/rfcs/issues/53, https://talk.peercoin.net/t/rfc-0026-max-tx-size-consensus-limit/16031
- Supersedes: N/A
- Superseded by: N/A
- Author: sandakersmann

## Summary

Add a Max Tx Size Consensus Limit to the protocol consensus rules.

## Conventions

- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Consideration

Bitcoin Core did not add a Max Tx Size Consensus Limit since they have MAX_BLOCK_SIGOPS_COST and a very low MAX_BLOCK_WEIGHT. When Peercoin wants to increase the MAX_BLOCK_WEIGHT in the future, it’s important that we have a Max Tx Size Consensus Limit in place to avoid excessively large transactions on the blockchain. We can not depend on Bitcoin Core to implement this since they have no intentions to increase MAX_BLOCK_WEIGHT.

## Detailed design

Add new consensus rule to `src/consensus/consensus.h` like Bitcoin Cash Node has done: https://github.com/bitcoin-cash-node/bitcoin-cash-node/blob/master/src/consensus/consensus.h

The Max Tx Size Consensus Limit should be put just above the current size of the biggest transaction in the blockchain.

## Advantages

* Avoid excessively large transactions on the blockchain

## Drawbacks

* Hardfork of protocol rules

## Alternatives

* Not increase MAX_BLOCK_WEIGHT

