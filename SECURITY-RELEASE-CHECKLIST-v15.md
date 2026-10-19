# Security release verification — v15

- [x] `node --check server.js`
- [x] `node --check security-smoke-test.js`
- [x] `node --check security-proxy-test.js`
- [x] `npm test` — all seven test programs passed.
- [x] Direct-message writes enforce block and friends-only policy server-side.
- [x] Reject malformed multi-party direct-conversation IDs and unknown message fields.
- [x] Reverse-proxy headers ignored unless the immediate peer matches `TRUST_PROXY_IPS`.
- [x] Regression test confirms production HTTPS cannot be bypassed with forged `X-Forwarded-Proto`.
- [x] ZIP integrity and SHA-256 manifest verified after packaging.

## Not certified

- No independent third-party audit or penetration test has been performed.
- No claim of parity with VK, Signal, or Telegram is made.
- The custom E2E design still lacks a reviewed Double Ratchet and full post-compromise security.
- Production deployment still requires correct TLS/proxy configuration, secrets, host isolation, backups and monitoring.
