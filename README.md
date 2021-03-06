# nocino

`nocino` is a simple asymmetric encryption mechanism

It is inspired by `fernet` (and leverages it internally), except with
asymmetric keys instead of symmetric ones.

## Rationale

You probably want `libsodium`, but you can't use `libsodium` because
it isn't packaged everywhere (notably RHEL and Ubuntu). Maybe you
can't use `libsodium` because there's a hard requirement for
NIST-approved primitives.

## Design considerations

This standard provides:

* efficient authenticated encryption,
* authenticated associated data,
* efficient support for multiple recipients,
* efficient asymmetric key rotation without having to re-encrypt,
* support for large (>64GB) messages (unlike AES-GCM-based
  constructions).

It does *not*:

* hide the public keys of the recipients,
* hide which capabilities are granted to which keys,
* specify canonicalization; it is possible for different ciphertexts
to represent the same message to the same recipients,
* prevent messages from being modified; specifically, it is possible
  to add or remove associated data, recipients (REVIEW: we can fix
  this; should we?)

It has the following considerations for implementors:

* an easy-to-parse format, where all fields are either fixed-width or
  length-prefixed, allowing a recognizer to parse the message in one
  pass;

## Message structure

A message is a [URL-safe base64][base64url] encoded sequence of bytes,
consisting of the concatenation of the representations of a sequence
of *packets*. All packets are [netstrings][netstring], to somewhat
mollify djb.

## Packets

Each packet is wrapped in a netstring.

### Authenticated ciphertext packet

The contents of the authenticated ciphertext packet are a
[`fernet`][fernet] message, except without the final URL-safe base64
encoding.

### Authenticated plaintext packet


### RSA encryption key packet

RSA encryption key packets contain the authentication key and the
encryption key, encrypted for a specific recipient using that
recipient's RSA key.

The contents are encrypted using OAEP.

### RSA MAC key packet

RSA encryption key packets contain the authentication key, encrypted
for a specific recipient using that recipient's RSA key.

The contents are encrypted using OAEP.

[oaep]: http://cseweb.ucsd.edu/~mihir/papers/oae.pdf
[base64url]: http://tools.ietf.org/html/rfc4648#section-5
[fernet]: https://github.com/fernet/spec/
