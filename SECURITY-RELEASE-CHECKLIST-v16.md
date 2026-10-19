# Security release checklist — v16 / 1.9.0

## Automated checks completed
- [x] `node --check server.js`
- [x] `node --check security-smoke-test.js`
- [x] `npm test` — all seven test programs pass
- [x] Direct messages reject plaintext `text` payloads even if a client sets `e: 2`
- [x] Gift endpoint respects blocks in both directions
- [x] Gift endpoint has a per-account burst limit
- [x] Shared failed-attempt limiter stores the refreshed attempt window correctly
- [x] Gift and bot-message IDs use cryptographic random suffixes
- [x] ZIP integrity verified after packaging
- [x] SHA-256 checksum generated for release archive

## Not completed / not certified
- [ ] Independent penetration test and remediation review
- [ ] Independent cryptographic protocol review
- [ ] Replace custom long-lived ECDH design with a maintained, reviewed E2E protocol including forward secrecy and post-compromise security
- [ ] End-to-end encryption for group conversations and attachments
- [ ] Multi-device security design and device/session management UI
- [ ] Production-scale shared storage, rate limits, monitoring, alerting, backup restore drills and disaster recovery validation
- [ ] Full OWASP ASVS 5.0 verification and evidence mapping

A green automated test suite proves only the tested behaviors. It does not certify parity with VK or any other service.
