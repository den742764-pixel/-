# Sirius Security v9 — release notes

## Changes

- Added a 7-day idle timeout for authenticated sessions alongside the existing 30-day absolute lifetime.
- Session activity timestamps are persisted at most once every five minutes to limit unnecessary database writes.
- Expired idle sessions have their server-side token hash rotated and their active streams closed.
- Long-lived Server-Sent Events connections are closed when the session expires or becomes idle for seven days.
- Media uploads reserve file IDs and per-user quota before asynchronous writes, preventing concurrent duplicate uploads from overwriting each other or double-counting quota.
- Media upload completion re-checks account existence, token validity, and session age; if the account was deleted or the token revoked during the write, the file is removed instead of being registered.
- Added integration regression coverage for idle-session expiry across restart and concurrent duplicate media uploads.

## Validation

Run `npm test`. The suite currently covers API and CSRF checks, account isolation, recovery/password rotation, media validation and deletion, E2E envelope integrity, key verification, absolute session expiry, idle session expiry, and duplicate upload races.

## Limitations

These tests are regression checks, not a substitute for an independent penetration test or cryptographic audit. The existing E2E protocol has not been replaced with a reviewed Signal Protocol implementation in this release.
- Media reads now require `Authorization: Bearer <session-token>` and no longer accept session tokens in query strings, reducing exposure through URL/access logs. The endpoint is restricted to GET.
