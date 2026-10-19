# Sirius v16 — release notes

Release: 1.9.0
Date: 2026-10-19

## Security fixes

1. **Direct-message plaintext downgrade blocked.** Creating a direct conversation message now requires the encrypted `e2e` envelope kind and a supported envelope version. A client cannot claim `e=2` while sending a `text` message and have the server accept it as an encrypted DM.
2. **Block enforcement is symmetric for gifts.** Sending a gift is rejected if either the sender blocked the recipient or the recipient blocked the sender. The previous endpoint checked only one direction and could bypass the recipient's block.
3. **Fixed failed-attempt limiter bookkeeping.** When refreshing an existing attempt window, the limiter now stores the filtered event list back into its map. Previously, subsequent attempts could append to a temporary array, preventing counters from accumulating as intended. This affected protections built on the shared failed-attempt tracker.
4. **Gift and order abuse throttling.** Gift sending is limited to 20 attempts per account per minute; order creation is limited to 10 attempts per account per minute. These are in-process limits; use a shared limiter and edge protection for multi-instance production deployments.
5. **Unpredictable generated IDs.** Server-generated gift and bot-message IDs now use cryptographic random bytes instead of `Math.random()` suffixes.
6. **Regression tests added** for plaintext DM rejection, recipient-side block enforcement on gifts, and the per-account gift rate limit.

## Validation performed

- `node --check server.js` — passed.
- `node --check security-smoke-test.js` — passed.
- `npm test` — all seven test programs passed, including the new regression assertions.

## Important security boundary

This release is a targeted hardening update, not proof that Sirius has the same overall security as VK. Direct-message encryption still lacks a reviewed Double Ratchet / forward secrecy, group conversations are not covered by the direct-message E2E protocol, and there has been no independent penetration test or cryptographic audit. Do not advertise audited parity until those items are completed and verified.
