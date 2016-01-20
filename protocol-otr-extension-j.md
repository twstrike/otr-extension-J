# Off-the-Record Messaging Protocol extension J

This document describes the J extension to the Off-the-Record protocol. It is based on the Off-the-Record Messaging Protocol version 3, that can be found at https://otr.cypherpunks.ca/Protocol-v3-4.0.0.html. This document only describes the additions to that protocol. The J extension is also backwards compatible using the version negotiation component of OTR - if only one peer supports J, the negotiation will simply choose version 3 instead.

The main goal of extension J is to update the protocol choices for OTR to more modern algorithms with larger security margins. The previous choices of DSA with 1024 bit keys and SHA-1 are at the end of 2015 becoming uncomfortably close to being possible to crack.

The main changes in this extension are to:

- Replace DSA long lived keys for signatures with Ed25519.
- Change the hash and HMAC algorithms from SHA-1 and SHA-256 into SHA3-256 in the OTR and SMP protocol

## Wire protocol

The J extension has a wire protocol that is completely compatible with previous versions of the OTR protocol. What that means is that previous correct implementations of OTR will be able to receive J extension messages and understand them well enough to avoid crashes. The biggest differences are in the representation of the new extension version in the various messages.

In the Query message it is now possible to use the letter J in addition with the other characters specified in the protocol, such that:

`"?OTRv23J?"`

is a likely thing to see on the wire.

The tagged plaintext message can now also contain:

`"\x20\x09\x20\x20\x09\x20\x09\x20"`

to indicate willingness to talk using the J extension.

In every message that includes a SHORT Protocol Version entry, this should be the unsigned short value 0xFE32 (the ASCII code for upper case J, added to 65000).

The fragmentation format is the same as for OTR version 3. That means you can't tell the difference between a 3 and a J protocol message from the fragmented pieces - you will have to wait until after reassembly to finalize how to deal with a message.

Finally, the version of the SMP protocol described in this protocol will stay version 1. That means that the SMP protocol version 1 inside of extension J is _not_ the same as SMP protocol version 1 inside of the OTR protocol version 2 or 3.

## Representation of keys

The only valid public and private keys used for signing when using extension J is Ed25519 keys. A public key is 32 bytes and a private key is 64 bytes where the last 32 bytes are the same as the public key.

In the various places where serialization of the public key is necessary, the format looks like this:

```
OTR public authentication Ed25519 key (PUBKEY):
    Pubkey type (SHORT)
        Ed25519 public keys have type 0x0003
    keydata (DATA)
        The data will always be 32 bytes, so using the DATA format is technically
        redundant, but should help keep the wire format simple
```

A signature will always have this format:

```
Ed25519 signature (SIG):
    (len will always be 64 bytes, or 512 bits)
    len byte unsigned data, little-endian
```

Fingerprints for extension J keys will be calculated by taking the SHA3-256 hash of the byte-level representation of the public key. That means a fingerprint using extension J is 96 bits longer than version 3 fingerprints - user interfaces should keep this in mind.

## Hashes

There are a number of places in OTR version 3 that uses SHA-1 or SHA2-256 in order to calculate message digests or message authentication codes. All of these places should use SHA3-256 when using extension J. Specifically, these places are:

- In the Authenticated Key Exchange:
    - In the D-H Commit Message, the hashed `g^x` parameter should be the SHA3-256 hash of the `gxmpi`, not the SHA-256 hash.
    - In the Reveal Signature Message, the `MB` parameter should be computed using SHA3-256(K||M) instead of SHA256-HMAC
    - In the Reveal Signature Message, the MAC'd signature should be computed using SHA3-256(K||M) instead of SHA256-HMAC-160.
    - In the Signature Message, the `MA` parameter should be computed using SHA3-256(K||M) instead of SHA256-HMAC
    - In the Signature Message, the MAC'd signature should be be computed using SHA3-256(K||M) of the encrypted signature field, not SHA256-HMAC-160.
- In the Data message:
    - The Authenticator should be calculated using SHA3-256(K||M), instead of SHA1-HMAC.
- Computing AES keys, MAC keys and the secure session ID:
    - Redefine `h1()` to use SHA3-256 instead of SHA1.
    - Redefine `h2()` to use SHA3-256 instead of SHA256.
    - The sending and receiving MAC keys should be calculated to be the output of SHA3-256 instead of SHA1
- When revealing MAC keys
    - Instead of revealing MAC keys by concatenating 20 byte values, concatenate 32 byte values (since the MAC key will be longer using extension J).


## Hashes in SMP

There are a number of places in the SMP protocol that use hashes. These places should all be replaced with SHA3-256, as follows:

- Creating the actual secret `x` or `y` should be done by taking the SHA3-256 hash of the material, not the SHA-256 hash.
- The SMP Hash function should be SHA3-256 instead of SHA-256 for every place where the hash of one or two MPIs is required.
- To calculate `c2` and `c3` use SHA3-256 instead of SHA-256.
- To calculate `cP` use SHA3-256 instead of SHA-256.
- To calculate `cR` use SHA3-256 instead of SHA-256.

## Various resolution mechanisms

This extension adds a new policy flag called ALLOW_EXTENSION_J - this works exactly the same as the ALLOW_V2 and ALLOW_V3 policies. OTR is also only disabled if all four of the ALLOW_ flags are disabled.

Extension J only supports Ed25519 keys - as such, it is not possible to use previously established DSA keys for extension J communication. This presents a bootstrap problem for peers with many verified fingerprints. This protocol specification does not specify an automated solution for this problem, although it is practical to turn off extension J, send the new fingerprint information for the Ed25519 keys and then turn on extension J again.

If extension J is implemented, it should take precedence over version 3 and version 2.

