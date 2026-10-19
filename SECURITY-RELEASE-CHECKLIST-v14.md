# Security release verification — v14

## Executed checks

- `node --check server.js` — passed.
- `node --check security-smoke-test.js` — passed.
- `npm test` — all six test programs completed successfully, including the message, crypto, key-verification, and session-expiry suites.
- ZIP integrity check is performed after packaging.

## Added in v14

- Message records are validated server-side before persistence and broadcast.
- Regression tests reject timestamps one day in the future, oversized receipt maps, and too-short recovery tokens.
- Package version bumped to 1.7.0.

## Security claims deliberately not made

- These tests are not a penetration test, formal verification, or independent audit.
- Image file validation checks signatures and size bounds; it does not fully decode and sanitize every image format.
- E2E remains the project's custom protocol and does not provide a reviewed Signal-style Double Ratchet or full post-compromise security.
- Production safety still depends on HTTPS, correct proxy trust configuration, strong secret management, OS/file permissions, backups, and deployment isolation.
- No numeric rating can be considered "ironclad" from regression tests alone.
