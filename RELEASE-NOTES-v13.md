# Sirius Security v13 — 2026-10-19

## Additional hardening

- **Bounded SSE output:** slow clients are disconnected when queued event-stream output would exceed 4 MiB. This limits memory consumed by clients that stop reading.
- **Bounded stream-ticket registry:** expired tickets are removed at capacity; if 5,000 live tickets remain, new tickets are rejected rather than growing the map indefinitely.
- **Security headers on early responses:** baseline browser-security headers are set before URL parsing, CSRF checks, OPTIONS handling, and HTTPS enforcement, so early error paths receive them too.
- **Request-target limits:** oversized request targets, paths, and queries are rejected with 414 before application routing.
- **Avatar/cover data validation:** profile image data URLs must pass the supported image-signature checks used for media uploads; arbitrary or mislabeled data URLs are rejected.
- Session policy unchanged: active sessions renew and expire after 365 days of inactivity.

## Validation

The full test suite and syntax checks are required before release. These regression checks do not replace an independent penetration test.
