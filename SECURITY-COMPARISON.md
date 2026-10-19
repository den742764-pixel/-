# Sirius security comparison — 2026-10-19

This is a source-level feature comparison, not a penetration-test certification. Sirius checks listed below were run against this release; Telegram/VK descriptions are based on their publicly documented behavior and independent published overviews.

## Sirius — this build

- Direct 1:1 message payloads are encrypted client-side with WebCrypto ECDH P-256, HKDF-SHA-256 and AES-256-GCM.
- New message envelope v2 uses fresh random HKDF salt per message and authenticates conversation ID, sender, recipient, message ID and timestamp as AAD.
- Public P-256 JWK validation, local peer-key pinning, changed-key send blocking, safety-code display, and clearing of persistent session tokens/plain private-key JWKs are implemented.
- Server enforces production HTTPS by default, strict JSON Content-Type, same-origin/explicit-allowlist Origin checks on browser POSTs, an in-memory per-IP POST cap without Redis, authentication on media access, one-use stream tickets, token revocation, and tested private-message access controls.
- Uploaded image data URLs are checked against PNG/JPEG/WebP/GIF byte signatures; unexpected internal errors are returned generically, and obvious developer-secret placeholders are rejected in production.
- Persistent login now uses an AES-GCM encrypted session bundle in IndexedDB with a non-exportable WebCrypto key; no persistent session token/private JWK is stored in localStorage. This reduces casual at-rest exposure, but does not protect against same-origin XSS.
- Automated tests cover server authorization regressions and E2E v2 encryption/decryption, metadata substitution and ciphertext tampering.
- Not implemented: reviewed Double Ratchet, per-message forward secrecy, post-compromise security, signed/one-time prekey bundles, key transparency, complete multi-device E2EE, client-side attachment encryption for all media/features, or an independent audit.

## Telegram

- Ordinary Cloud Chats use client-server/server-client encryption and are stored in Telegram Cloud; they are not end-to-end encrypted by default.
- Secret Chats use end-to-end encryption, are device-specific, and Telegram documents key comparison and periodic re-keying/PFS.
- Telegram has a mature, deployed protocol and client ecosystem, extensive operational security work, and public protocol documentation. Its architecture and threat model differ from Sirius.

Official references:
- https://www.telegram.org/faq#q-do-i-need-to-trust-telegram-for-this-to-be-secure
- https://core.telegram.org/api/end-to-end
- https://core.telegram.org/api/end-to-end/pfs

## VK Messenger / standard VK direct messages

- Public independent overviews describe ordinary VK Messenger chats as transport-encrypted, not end-to-end encrypted by default; server-side services can access message content. The precise behavior can vary by product/version, and this comparison is not based on a VK cryptographic protocol audit.
- A third-party browser extension can add its own E2EE layer, but that is not the same as native, default E2EE in VK Messenger.

References:
- https://www.anti-malware.ru/analytics/Market_Analysis/Popular-messengers-in-Russia
- https://cheburchat.com/ (third-party project; its claims are not VK's official documentation)

## Bottom line

For ordinary 1:1 message content, Sirius now has an actual client-side E2EE layer, whereas standard Telegram Cloud Chats and standard VK Messenger chats are not documented as E2EE by default. That does **not** make Sirius equivalent to Telegram Secret Chats: Sirius lacks a reviewed ratcheting protocol, forward secrecy, robust device/prekey management, key transparency, comprehensive attachment/group E2EE, and independent audit. ## Overall approximate rating (not a certification)

On a 10-point scale for overall practical security, considering cryptography, maturity, operations, and audit evidence: **Sirius 5/10; Telegram overall 7/10; Telegram Secret Chats 8/10; standard VK Messenger chats 4/10.** These are judgment-based estimates, not measured benchmark scores. Sirius gets credit for actual client-side encryption, key-change blocking, CSRF/Origin controls, upload-signature checks, and access-control tests, but is reduced by its custom unaudited protocol, lack of ratchet/forward secrecy, incomplete media/group coverage, and limited operational maturity. The new HTTP hardening does not justify a 7/10 score by itself. Telegram Secret Chats are more mature and document periodic re-keying/PFS; Telegram Cloud Chats are not E2EE. VK score refers only to standard chats and the limited public evidence cited above.

No score establishes that a service is safe against every attacker, and no independent penetration test of Sirius has been performed.


## v11 note
The v11 hardening adds bounded in-memory failed-attempt tracking, clean 413 handling for oversized bodies, and a regression fix for removed group members editing old messages. These changes improve abuse resistance and authorization correctness but do not materially close the cryptographic gap described above; the overall rating should not be raised to 7/10 on this basis alone.


## Версия v15 — статус сравнения

В v15 добавлены серверные проверки блокировок и режима «только друзья» для новых личных сообщений, строгий формат ID личного диалога и проверка доверенного reverse proxy через `TRUST_PROXY_IPS`. Это повышает защищённость API, но **не даёт оснований объявлять Sirius равным VK**: неизвестны внутренние детали VK, а независимого аудита Sirius и рецензирования криптографического протокола нет. Сравнивать можно только по конкретным проверенным контролям, а не по абсолютному числу.
