# Sirius Security v15 (1.8.0)

## Server-side direct-message privacy enforcement

- New direct messages must use a canonical two-account conversation ID; forged multi-party IDs are rejected.
- The server enforces the recipient's friends-only messaging preference and both users' block relationship, rather than relying on the UI to enforce it.
- Group IDs matching the direct-message ID format are rejected to avoid namespace collisions between group conversations and private dialogs.
- Message records accept only the documented fields; arbitrary extra server-controlled fields are rejected.

## Trusted reverse-proxy hardening

- `X-Forwarded-Proto` and `X-Forwarded-For` are trusted only if the immediate TCP peer matches `TRUST_PROXY_IPS`.
- `TRUST_PROXY_IPS` accepts comma-separated exact IP addresses or CIDR ranges; `TRUST_PROXY` still specifies the number of trusted proxy hops.
- With no proxy allowlist, forwarded headers are ignored. This prevents a direct client from spoofing HTTPS or evading IP-based request limits by forging forwarded headers.
- Added regression coverage for both untrusted and explicitly allowlisted proxy behavior.

## Verification

- `node --check server.js`, `node --check security-smoke-test.js`, and `node --check security-proxy-test.js` passed.
- `npm test` passed all existing crypto/session suites plus new DM policy, forged multi-party conversation, schema and reverse-proxy regression checks.
- Session inactivity remains one year, sliding with activity; active users are not forced to re-login every 30 days.

## Limitations

This is targeted hardening, not a claim of security parity with VK and not an independent penetration test. The custom E2E protocol still lacks a reviewed Double Ratchet / full post-compromise security, and this release does not replace an independent audit or production infrastructure review.
