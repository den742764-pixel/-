# Release checklist — v22

- [x] Syntax-check every server-side JavaScript test and source file.
- [x] Parse inline application JavaScript.
- [x] All prior security, crypto, session, proxy, privacy, SSE and TCP tests pass.
- [x] Live server starts with isolated temporary data directory.
- [x] `/api/ping` returns HTTP 200 and `{ "ok": 1 }`.
- [x] Main HTML returns HTTP 200 with expected security headers.
- [x] CSP script hashes match inline scripts.
- [x] Client API route literals map to server handlers.
- [x] Protected API rejects unauthenticated request; wrong content type and unknown route are rejected.
- [ ] Verify calls on two actual devices and real TURN relay.
- [ ] Verify browser/Android push in the target deployment.
- [ ] Configure production HTTPS, secret `DEV_CODE`, data persistence, backups and proxy trust list.
- [ ] Independent cryptographic audit and reviewed Double Ratchet migration remain outstanding.
