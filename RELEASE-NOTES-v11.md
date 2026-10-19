# Sirius Security v11 — 2026-10-19

## Changes

- Session lifetime remains sliding: active sessions are renewed, and a token expires after 365 days without activity. Logout, password change and recovery still revoke old tokens.
- Failed-attempt tracking is now memory-bounded (20,000 keys; at most 100 timestamps per key) and prunes stale entries. This reduces the risk of memory exhaustion from requests using many unique IP/account identifiers. The limiter remains process-local unless Redis is configured.
- Oversized HTTP request bodies are rejected with HTTP 413. The server drains the request rather than destroying the socket before the response can be sent.
- Group message mutation now checks current membership; a user removed from a group cannot edit their old messages. In channels, only current administrators can edit messages.
- Regression tests added for clean oversized-body rejection and write revocation after group removal.

## Validation

- `node --check server.js`
- `node --check security-smoke-test.js`
- `npm test` — all 16 reported regression checks passed, including crypto envelope integrity, key-verification fail-closed behavior, session expiry/renewal, media authorization, and cross-account isolation.

## Security limitations remain

This release does not add a reviewed Signal Protocol/Double Ratchet, per-message forward secrecy, complete multi-device E2EE, universal encrypted attachments, MFA/passkeys, distributed rate limiting without Redis, or an independent penetration test. Do not interpret the test results as certification or claim a verified 7/10 rating.
