# May 2018 Hardfork Specification

Version Draft 0.2, 2018-02-01

## Summary

When the median time past[1] of the most recent 11 blocks (MTP-11) is greater than or equal to UNIX timestamp 1526400000 Bitcoin Cash will execute a hardfork according to this specification. Starting from the next block these consensus rules changes will take effect:

* Blocksize increase to 32 MB
* Re-enabling of several opcodes
* Transaction version enforcement
* Creation of transaction version 3

The following are not consensus changes, but are recommended changes for Bitcoin Cash implementations:

* Automatic replay protection for future hardforks
* Increase OP_RETURN relay size to 223 total bytes
* Create only version 3 transactions and reject transactions > 3 but do not ban the sender

## Blocksize increase

The blocksize hard capacity limit will be increased to 32MB (32000000 bytes).

## OpCodes

Several opcodes will be re-enabled per [may-2018-opcodes](may-2018-opcodes.md).  Any transaction that contains these opcodes MUST be marked version 3.

## Transaction Version

After the May 2018 Hardfork:
 * Miners will enforce transaction versions.  A post-fork block is invalid if it contains a transaction of version 4 or larger.
 * Miners will enforce transaction content.  A version 1 or 2 transaction MUST not contain any of the new or re-enabled opcodes described in this document.  A post-fork block with such a transaction is invalid.

### Transaction Versioning Rationale

The transaction version is currently considered nonstandard (not relayed) if it is not 1 or 2.  However, the transaction version is currently not a consensus parameter -- it is possible for miners to mine transactions with nonstandard versions.  This makes the transaction version less useful because there is no consensus as to what a particular transaction version means.  Enforcing transaction versioning will provide the following benefits:

 * Replay Protection
 
A user can create a version 3 transaction even if the transaction does not use any version 3 specific opcodes.
After the May 2018 Hardfork, this provides weak replay protection since version 3 transactions are not relayed, but still could be mined.
After subsequent hardforks and subsequent transaction version increases, a version N>3 transaction will be illegal on the "legacy" May 2018 Hardfork chain.  This provides strong replay protection since the "legacy" chain would also need to hard fork to accept them.  The only way to stop replay on a chain that deliberately hard forks to be compatible with your hard fork is to mix newly-minted coins into every transaction.  Since this is onerous on users, requiring that the legacy chain also hard fork is a reasonable solution.

 * Reduction in Attack Surface
 
If new opcodes are added but the transaction version is not changed, Bitcoin Cash clients need to execute or at least parse the script to determine whether it can be understood.  This opens the possibility that a new opcode inadvertently triggers a latent bug in some client.  A simple check of the transaction version can eliminate this class of accidents.  Additionally, clients can distinguish whether a connected client is attempting to exploit a script interpreter bug (deliberately sending a bad script), or is just sending a newer script version.  This will allow the client to ban attacking clients but maintain connections with upgraded clients.


## Automatic Replay Protection

When the median time past[1] of the most recent 11 blocks (MTP-11) is greater than or equal to UNIX timestamp 1542300000 Bitcoin Cash full nodes implementing the May 2018 consensus rules SHOULD enforce the following change:

 * Update `forkid`[1] to be equal a non-zero unique constant equal to 0xFF0001.  ForkIDs beginning with 0xFF will be reserved for future protocol upgrades.

This particular consensus rule MUST NOT be implemented by Bitcoin Cash wallet software.

[1] The `forkId` is defined as per the [replay protected sighash](replay-protected-sighash.md) specification.
