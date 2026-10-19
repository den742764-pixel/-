# Sirius v24 — Support bots and Premium

- Adds two pinned local assistant chats for every signed-in user: Support and Stars/Premium.
- Support bot stores requests server-side; the owner can review open tickets and reply from the owner panel. Replies arrive in the existing system notification chat.
- Adds 30-day Sirius Premium (500 ★) and gifting Premium to another username (650 ★), both enforced server-side.
- Premium benefits: public diamond badge, profile Premium marker, and dedicated Premium shop screen.
- Server tracks expiry timestamps, deducts stars atomically in the single-threaded request handler, and rejects insufficient balance / unknown recipients.
- Adds regression tests for bot ticket flow, Premium purchase/gifting, and repeat/invalid requests.

Premium is currently purchased using stars; stars top-ups remain manually confirmed by the owner. The bots are built-in app chats, not independent external Telegram bots.
