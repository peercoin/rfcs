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

- 5 bytes application identifier
- 15 "burn identifier" bytes: `7bdef7bdef7bdef7bdef7bdef7bdef`
- 12 bytes application specific data

### Application Identifier

The witness program shall start with the application identifier. This will allow
the start of the bech32 address to begin with a recognisable application prefix.
The identifiers shall be 5 bytes that are encoded into 8 characters. For
example, `c9bbb0ff2f` will encode into `example0`.

It is recommended to pad the encoded identifier with `0` characters to the
right, when less than 8 identifying characters are desired. The bytes
`7bdef7bdef` will encode into `00000000`.

It is recommended that the application identifiers are unique between different
applications to avoid conflicts.

### Burn Identifier

The 15 bytes following the application identifier shall be
`7bdef7bdef7bdef7bdef7bdef7bdef` which shall encode into 24 `0` characters.
These bytes shall be used to identify burn addresses. The `0` characters make
the burn addresses easily recognisable.

Because these 15 bytes are fixed, it is not possible to brute force an
associated spendable script (pre-image). 15 bytes corresponds with 120 bits. The
current bitcoin hash rate corresponds to around 67 bits per second of
brute-force and wouldn't reach 120 bits, even after 10 million years.

### Application Specific Data

The final 12 bytes are for application specific data. Applications can identify
addresses by the application ID and burn identifier, and then use the
application data for whatever purpose. It is recommended to pad this data to the
left with `0` characters, using the same repeating 5-bit pattern:
`7bdef7bdef7bdef7bdef7bde`.

For example, if 3 bytes of data are required to be `010203` then this data can
packed as `7bdef7bdef7bdef7bd010203` with the required data on the right. This
encodes as `00000000000000gpqgps`.

### Example Address

If an address was to use `pc1qexample0` as a prefix and `010203` as hex data,
then the bytes shall be as follows:

    c9bbb0ff2f7bdef7bdef7bdef7bdef7bdef7bdef7bdef7bdef7bdef7bd010203

This encodes into the following address (using the testnet hrp):

    tpc1qexample000000000000000000000000000000000000000gpqgpslnr273

If the data was to include 12 bytes of application data using
`0102030405060708090a0b0c`, then the address would look like:

    tpc1qexample0000000000000000000000000qypqxpq9qcrsszg2pvxqd4vc49

### Exclusion from UTXO Set

If a 32 byte P2WSH witness program has the 15 byte burn identifier from index 5,
this shall be identified by the Peercoin protocol as being "unspendable". Any
output containing such a program will not be added to the UTXO set. These
outputs may be identified in the `CScript::IsUnspendable` method, but care must
be taken to ensure this function is used against output scripts only.

The chain state database will need to be updated once to remove any existing
outputs with burn witness programs.

## Drawbacks

To avoid UTXO bloat, the scheme requires changes to the wallet software to
filter these outputs from the UTXO set. However this may be deferred to a future
date, even after the addresses are being used.

12 bytes of application specific data is a small amount and may not be suitable
for all applications, however OP_RETURN is a possible alternative where more
data is required.

The `0` characters may be hard to manually type as the user would have to count
them, but most of the time it is expected that the user will use a QR code or
copy the address.

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

15 zero bytes may be considered too much. A thousand years worth of bitcoin
mining at the current hash rate is equal to around 100 bits of SHA-256 hashes.
The zero bytes could be reduced, possibly as low as 12 bytes (96 bits). If there
ever was a spendable script pre-image found, this would be rendered undependable
by the new protocol rule in any case and accidental collisions are very
unlikely.

The 5 byte application ID could also be reduced to 2 (as with port numbers), 3
or 4 bytes, though this increases the chance of accidental ID collisions or
running out of ID space. The length of the prefix characters would be shorter,
allowing for less recognisable words.

If the zero bytes were reduced to 12 bytes and the application ID down to 2
bytes, this would leave 18 bytes of application data.

