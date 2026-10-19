# Sirius Security v12 — 2026-10-19

## Security hardening

- **Profile write allowlist:** authenticated clients can change only explicitly supported profile fields. Server-managed fields such as phone/hidden flags and arbitrary privilege-like properties are rejected. Field types and lengths are checked, including booleans, public-key JWKs, timestamps, and image/cover strings.
- **Group invariants:** member and administrator lists are validated. Membership must be unique and refer to existing accounts; administrators must be members; the group owner must remain a member and administrator. Invalid group-role changes are rejected.
- **Account deletion cleanup:** deleting an account removes its owned groups and their messages, account-linked orders, recovery/session records, user-owned media, and audit rows tied to the account. A non-identifying deletion audit event is retained.
- **Request parsing:** malformed request targets and ambiguous/invalid `Content-Length` framing are rejected with HTTP 400 instead of being passed to application handlers.
- **Rate limiting:** general POST traffic now uses a bounded per-IP counter map that correctly supports the configured 600 requests/minute. Previously, reusing the failed-attempt event store (capped at 100 events per key) meant that nominal 600/minute threshold could never be reached. The counter map is capped at 20,000 keys and stale entries are pruned.
- **Regression tests:** added checks for profile privilege-field writes, server-managed profile fields, type validation, invalid group administrator/member lists, and deletion of owned group conversations and account credentials.
- Session policy unchanged: active sessions renew; inactivity for 365 days expires the token.

## Validation performed

- `node --check server.js`
- `node --check security-smoke-test.js`
- `npm test` — all 16 reported regression checks passed, including E2E envelope tamper rejection, first-contact key verification, cross-account isolation, media access control, account deletion, and session expiry/renewal.

## Limitations

This is not an independent security audit or a proof of vulnerability-free code. The client still does not implement a reviewed Signal Protocol/Double Ratchet with forward secrecy and post-compromise security; multi-device and attachment encryption are not comprehensively covered. Distributed rate limiting requires Redis. A real-world penetration test and dependency/configuration review remain separate from these regression tests.
