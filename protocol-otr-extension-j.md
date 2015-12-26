# Off-the-Record Messaging Protocol extension J

This document describes the J extension to the Off-the-Record protocol. It is based on the Off-the-Record Messaging Protocol version 3, that can be found at https://otr.cypherpunks.ca/Protocol-v3-4.0.0.html. This document only describes the additions to that protocol. The J extension is also backwards compatible using the version negotitation component of OTR - if only one peer supports J, the negotiation will simply choose version 3 instead.

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

Finally, the version of the SMP protocol described in this protocol will stay version 1. That means that the SMP protocol version 1 inside of extension J is _not_ the same as SMP protocl version 1 inside of the OTR protocol version 2 or 3.

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

A private key will sometimes be serialized in configuration files. The format for that will be the following, encoded as base64:

```
OTR private authentication Ed25519 key (PRIVKEY):
    Key type (SHORT)
        Ed25519 private keys have type 0x0003
    public keydata (DATA)
        The data will always be 32 bytes
    private keydata (DATA)
        The data will always be 64 bytes. This format repeats the public key data
        twice - this is done to match the typical in-memory representation of keys
```

A signature will always have this format:

```
Ed25519 signature (SIG):
    (len will always be 64 bytes, or 512 bits)
    len byte unsigned data, big-endian
```

Fingerprints for extension J keys will be calculated by taking the SHA3-256 hash of the byte-level representation of the public key. That means a fingerprint using extension J is 96 bits longer than version 3 fingerprints - user interfaces should keep this in mind.

## Hashes

There are a number of places in OTR version 3 that uses SHA-1 or SHA2-256 in order to calculate message digests or message authentication codes. All of these places should use SHA3-256 when using extension J. Specifically, these places are:

- In the Authenticated Key Exchange:
    - In the D-H Commit Message, the hashed `g^x` parameter should be the SHA3-256 hash of the `gxmpi`, not the SHA-256 hash.
    - In the Reveal Signature Message, the `MB` parameter should be computed using a SHA3-256-HMAC instead of a SHA256-HMAC
    - In the Reveal Signature Message, the MAC'd signature should be the SHA3-256-HMAC-160 of the encrypted signature field, not the SHA256-HMAC-160.
    - In the Signature Message, the `MA` parameter should be computed using a SHA3-256-HMAC instead of a SHA256-HMAC
    - In the Signature Message, the MAC'd signature should be the SHA3-256-HMAC-160 of the encrypted signature field, not the SHA256-HMAC-160.
- In the Data message:
    - The Authenticator should be calcuated using SHA3-256-HMAC-160, instead of SHA1-HMAC - note the truncation to 160 bits. This is necessary to keep the wire format compatible with version 3.
- Computing AES keys, MAC keys and the secure session ID:
    - Redefine `h2()` to use SHA3-256 instead of SHA256
  

## Hashes in SMP

## Various resolution mechanisms

(allowJ shouldn't happen if there is no ed25519 key available)

## Summary
