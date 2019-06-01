# Cold Minting

- Status: proposed ("proposed", "agreed", "active", "implemented" or "rejected")
- Type: new feature
- Related components: (if any)
- Start Date: 08-05-2014
- Discussion: https://talk.peercoin.net/t/cold-storage-minting-proposal/2336 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: (your name)

## Summary

The main idea is the use a new kind of address very similar to multi-signature addresses with two public keys, one that allows only minting and the other that allows only spending.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

Cold minting will increase minter participation in the network. It will allow minters to leave their spend keys in a safe cold storage while actively minting with their mint key on a hot system.
It should be expected that cold minting of this form will result in mint pools where a single operator is in control of many users' mint keys.
This proposal may also allow user-friendly multisig minting where multiple signatures are required to spend the coins but only the mind signature is needed to mint them.

## Detailed design

We add a new script opcode OP_COINSTAKE that pushes 1 if we’re in a coinstake transaction and 0 otherwise. It’s easy to do because we have access to the transaction when the script is run.
We allow a new standard script that authorizes one key if the transaction is a CoinStake, and the other one in all other situations.
The script could look like this:

OP_DUP

OP_HASH160

OP_COINSTAKE

OP_IF

mintingAddress.GetHash160()

OP_ELSE

spendingAddress.GetHash160()

OP_ENDIF

OP_EQUALVERIFY

OP_CHECKSIG

When a CoinStake transaction is generated, if the kernel is a cold minting address and we know the minting private key then we sign the transaction with this key.
Right now when you find a PoS block you can send the reward and the coins to another address. We must prevent that when a cold minting address is used: the coins and the reward must be sent to the cold minting address. We must also force the output value to be exactly the sum of the inputs + the reward (currently the protocol allows a lower value).
We make the block signature and validation also work when the kernel is a minting address (right now it only supports pay-to-pubkey outputs).
In the GUI we add a new button “New cold minting address” in the “Receive coin” tab. It opens a form asking for the minting address, the spending address and an optional label.
In the RPC server we add a new command “addcoldmintingaddress []”.
We add a new “Minting only balance” in the overview page that displays the amount of coins you cannot spend but you can mint with. This amount should not appear in the balance.
During the CoinStake creation we change the coin selection process to also select the outputs with which we can only mint.

## Drawbacks

By design, the mint keys are considered less important than the spend keys (i.e. they are held hot, given to a pool operator, etc).
This can be considered detrimental to network security because thieves or pool operators may gather mint keys to accomplish a double spend or other network attack.
In the case of mint pools, a pool operator may intentionally withold blocks in order to time a chain reorg for nefarious purposes without the spend key owner ever knowing.

## Alternatives

Sunny King’s cold locked transactions (https://bitcointalk.org/index.php?topic=194054.0) using a 'cold lock' transaction.

NXT leased forging (http://wiki.nxtcrypto.org/wiki/Glossary/fr#Leased_Forging) which allows other addresses to mint on your behalf for a given period of time.

d5000's coinage messaging (http://www.peercointalk.org/index.php?topic=2467.msg21919#msg21919) where you transfer coinage to another address.

Allowing the mint key to spend the reward to discourage giving out the mint key too freely (http://www.peercointalk.org/index.php?topic=2467.msg21197#msg21197)

## Unresolved questions

Is it worth it to increase PoS difficulty at the potential cost of minters not being fully engaged with the minting process?
