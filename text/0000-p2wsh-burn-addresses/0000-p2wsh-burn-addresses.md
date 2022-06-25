# P2WSH Burn Addresses

- Status: proposed 
- Type: new feature 
- Start Date: 25-06-2022 
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this) 
- Author: MatthewLM

## Summary

Applications may desire a mechanism to burn coins. This RFC describes a
standardised and backwards compatible scheme to produce burn addresses. Outputs
to these addresses can be filtered from the UTXO set similarly to OP_RETURN
outputs.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Applications such as immutable.place require that coins are burned as part of
their design. While it is possible to create burn addresses already, they
pollute the UTXO set. A standard way to identify burn addresses and outputs
allows them to be removed from the UTXO set and the associated supply.

## Detailed design

The address scheme will reuse the P2WSH (Pay to Witness Script Hash) output type
and associated bech32 addresses to be fully backward compatible with existing
wallets that support P2WSH.

Burn addresses shall encode a 32 byte "burn witness program" with the following
concatenated data:

- 16 zero bytes
- 4 bytes application identifier
- 12 bytes application specific data

The witness program will start with 16 zero bytes to ensure that it is not
possible to brute force an associated spendable script (pre-image). 16 bytes
corresponds with 128 bits. The current bitcoin hash rate corresponds to around
67 bits per second of brute-force and would not come close to 128 bits, even
after a thousand years.

4 bytes are used for the application identifier. This may be anything, but
should be unique between different applications to avoid conflicts.

If a 32 byte P2WSH witness program starts with 16 bytes, this shall be
identified by the Peercoin protocol as being "unspendable". Any output
containing such a program will not be added to the UTXO set. These outputs may
be identified in the `CScript::IsUnspendable` method, but care must be taken to
ensure this function is used against output scripts only.

The chain state database will need to be updated once to remove any existing
outputs with burn witness programs.

## Drawbacks

To avoid UTXO bloat, the scheme requires changes to the wallet software to
filter these outputs from the UTXO set. However this may be deferred to a future
date, even after the addresses are being used.

12 bytes of application specific data is a small amount and may not be suitable
for all applications, however OP_RETURN is a possible alternative where more
data is required.

## Alternatives

It is possible to burn coins using OP_RETURN, but OP_RETURN outputs are not
widely supported across wallets and do not have an address format to facilitate
easy payments.

Fees can be burnt, but fees are not associated with particular addresses. Not
all wallets may allow configurable fees, or this process may involve more steps
for the user. Burning to an address enables easy accounting, makes the payment
process clear to the user, and allows burn outputs to be included in a
multi-output transaction that may have multiple burn outputs.

It is already possible to create undependable "burn" addresses but providing a
standard scheme allows addresses to be identified and outputs removed from
the UTXO set. This reduces the number of outputs within the UTXO set and adjusts
the `total_amount` supply figure returned from the `gettxoutsetinfo` RPC
command.

## Unresolved questions

When updating the wallet software there are three options to deal with
pre-existing burn outputs:

1. Automatically update the chain state database to remove these outputs.
2. Ask users to reindex the blockchain.
3. Accept that the UTXO set would be inconsistent between wallets but this has
   no impact on consensus as these outputs are unspendable in any case (unless
   SHA-256 is broken to 128-bits).

16 zero bytes may be considered too much. A thousand years worth of bitcoin
mining at the current hash rate is equal to around 100 bits of SHA-256 hashes.
The zero bytes could be reduced, possibly as low as 12 bytes (96 bits). If there
ever was a spendable script pre-image found, this would be rendered undependable
by the new protocol rule in any case and accidental collisions are very
unlikely.

The 4 byte application ID could also be reduced to 2 (as with port numbers) or 3
bytes, though this increases the chance of accidental ID collisions or running
out of ID space.

If the zero bytes were reduced to 12 bytes and the application ID down to 2
bytes, this would leave 18 bytes of application data.

