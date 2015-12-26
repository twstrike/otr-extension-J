# Off-the-Record Messaging Protocol extension J

This document describes the J extension to the Off-the-Record protocol. It is based on the Off-the-Record Messaging Protocol version 3, that can be found at https://otr.cypherpunks.ca/Protocol-v3-4.0.0.html. This document only describes the additions to that protocol. The J extension is also backwards compatible using the version negotitation component of OTR - if only one peer supports J, the negotiation will simply choose version 3 instead.

The main changes in this extension is:

- Enable Ed25519 long lived keys for signatures
- Change the hash and HMAC algorithms from SHA-1 and SHA-256 with SHA-3 with 256 bits in the OTR and SMP protocol

## Wire protocol

The J extension has a wire protocol that is completely compatible with previous versions of the OTR protocol. What that means is that previous correct implementations of OTR will be able to receive J extension messages and understand them well enough to avoid crashes. The biggest differences are in the representation of the new extension version in the various messages.

In the Query message it is now possible to use the letter J in addition with the other characters specified in the protocol, such that:

  "?OTRv23J?"

is a likely thing to see on the wire.

The tagged plaintext message can now also contain:

  "\x20\x09\x20\x20\x09\x20\x09\x20"

to indicate willingness to talk using the J extension.

In every message that includes a SHORT Protocol Version entry, this should be the unsigned short value 0xFE32 (the ASCII code for upper case J, added to 65000).

Finally, the version of the SMP protocol described in this protocol will stay version 1. That means that the SMP protocol version 1 inside of extension J is _not_ the same as SMP protocl version 1 inside of the OTR protocol version 2 or 3.

## Representation of keys

(fingerprints)

## Various resolution mechanisms

## Hashes

## Hashes in SMP

## Summary
