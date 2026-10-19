# Sirius Security v10 — rolling session expiry

## Changes

- Removed the 30-day absolute session lifetime and 7-day idle timeout.
- Sessions now expire after 365 days without activity. Valid activity refreshes both the last-activity timestamp and the expiry deadline, so an active user is not forced to log in again every 30 days or every year.
- Existing session metadata is migrated to the rolling one-year inactivity window without rotating valid tokens during deployment.
- Expired sessions are revoked server-side; existing protections still revoke sessions on logout, cloud-password changes, and account recovery. SSE connections and pending media writes check current session validity.
- Bounded and normalized recovery UIDs before using them as rate-limit keys, and bounded token/proof inputs; oversized malformed identifiers cannot create unbounded per-identifier limiter keys.
- Added integration regression tests for expiry after one year of inactivity, renewal of an active session across server restarts, and oversized recovery identifiers.

## Validation

`node --check server.js` and `npm test` pass. Tests cover API/CSRF checks, account isolation, recovery/password rotation, media validation and deletion, E2E envelope integrity, key verification, one-year inactivity expiry, rolling renewal, and malformed recovery-input handling.

## Security scope

This release improves session continuity and expiry behavior; it does not itself raise cryptographic assurance to a certified score. The E2E protocol remains custom and has not been independently audited or replaced with a reviewed Signal Protocol implementation. Do not describe Sirius as independently audited or equivalent to Signal.
