# Sirius cryptography status and required migration

## Current verified code state

The browser currently uses Web Crypto ECDH P-256, derives a reusable AES-GCM key from the long-lived account key pair, and encrypts each message with a fresh random 96-bit IV. The server stores ciphertext for messages marked `e: 1`; it does not decrypt those message bodies.

## Important limitation

This is not equivalent to a reviewed Double Ratchet protocol. A long-lived recipient private key and reusable per-peer key mean a later compromise of that key may expose stored historical ciphertext. Key changes, device enrollment, identity-key verification, replay/order handling, and robust group key management are not a complete ratchet implementation. The existing numeric “security code” is a fingerprint of public keys, not proof that the peer key was obtained authentically.

## Required production migration

1. Choose a maintained, independently reviewed implementation of the Signal Protocol / Double Ratchet that supports the intended browser and server environment; do not invent a new ratchet in application code.
2. Add per-device identity keys, signed prekeys, one-time prekeys, and a transparent identity-key-change warning. Authenticate identity keys out-of-band or with a reviewed key-transparency design.
3. Implement the full session state machine, including skipped message keys, replay rejection, out-of-order delivery, device revocation, and backup/export semantics.
4. For groups, use a reviewed group protocol such as MLS or the library’s supported group protocol; do not reuse one static group key.
5. Add known-answer tests, tamper/replay tests, multi-device tests, key rotation tests, and a migration plan for existing encrypted messages.
6. Obtain a specialist cryptographic review before claiming forward secrecy or post-compromise security.

Do not claim “Telegram-level”, forward secrecy, or post-compromise security until these steps are implemented and independently reviewed.
