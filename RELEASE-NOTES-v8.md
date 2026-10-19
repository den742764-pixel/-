# Sirius Security v8

## Account and session hardening

- Added persisted absolute session expiration (30 days) to reduce the lifetime of a stolen session token.
- New registration, successful account recovery/login, and cloud-password changes issue fresh session metadata.
- Logout removes session metadata; password changes and recovery rotate the token and close existing streams.
- Existing accounts are migrated with a bounded 30-day session lifetime instead of indefinite sessions.
- Expired sessions are rejected after restart; integration regression test modifies persisted expiry and verifies rejection.

## Verification

Run `npm test`. The suite includes the existing account authorization, media, E2E envelope, and safety-key checks plus an integration test for persisted session expiry.

## Limitations

This is a hardening release, not an independent security audit. Session expiry does not provide per-device session management or multi-factor authentication. The existing custom E2E protocol still lacks a reviewed Double Ratchet and full forward secrecy.
