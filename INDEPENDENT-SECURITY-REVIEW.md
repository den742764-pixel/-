# Sirius — security review and prioritized remediation

Date: 2026-10-19
Scope: static review of `server.js`, `index.html`, `security-smoke-test.js`, package/config files, plus execution of the included smoke tests. This is **not** a third-party penetration test or a production-host review.

## Verified in this environment

- `node --check server.js` passed.
- `node --check security-smoke-test.js` passed.
- `npm test` passed all six reported checks: no implicit account bootstrap, server-assigned ID/duplicate username rejection, one-use stream ticket/replay rejection, logout token revocation, anonymous media denial, and media deletion with account deletion.
- Registration and recovery attempt limits exist in process memory; per-user write and signaling limits exist.
- Browser code uses Web Crypto ECDH P-256, HKDF-SHA-256 and AES-GCM for some personal message bodies; recovery uses PBKDF2-SHA-256 with 250,000 iterations and an encrypted private-key vault.

## Findings that prevent a Telegram-level security claim

### High — session and private-key material in browser localStorage

The client persists the session token and exported private key (`sid`, `stok`, `spk`) in `localStorage`. Any successful same-origin script injection or malicious extension with page access can steal both. The private key is exported as JWK and marked extractable. CSP helps but does not make localStorage safe against XSS. Consider a threat-model-driven redesign: non-extractable CryptoKey where feasible, safer session architecture, strong XSS prevention, and device-specific encrypted key storage. A web app cannot fully protect secrets from a compromised browser/device.

### High — no forward secrecy / ratcheting in the message protocol

The reviewed personal-message path derives a long-lived shared AES key from static account ECDH keys and caches it. There is no visible Double Ratchet, per-message key evolution, or key deletion strategy. Compromise of a long-term private key can therefore expose recorded messages encrypted under the same static key. A code-security review alone cannot substitute for a reviewed protocol implementation. Adopt a mature, audited protocol/library and versioned message format rather than designing a custom ratchet.

### High — public-key authenticity and replacement are not enforced

A profile's public key is distributed by the server and used for ECDH. A safety-code display exists, but the flow does not appear to enforce verification or warn on key changes before continuing. A malicious server or account takeover could replace a key and facilitate a man-in-the-middle attack. Add persistent key fingerprints, key-change warnings, identity/device keys, signed prekeys and a documented verification flow. Test active key substitution.

### High — server-side authorization needs broader negative testing

`state()` serializes every record permitted by `canSee()`, while authorization rules are custom per collection. Public profile fields include `pk`; group visibility and message membership use several compact rules. The current smoke tests do not systematically test cross-account reads/writes, forged sender IDs, group admin escalation, friend/block edge cases, replay, or malformed nested objects. Add a table-driven two-user/two-device authorization suite and fuzz the API. Treat this as an unverified risk, not proof of a confirmed exploit.

### High — distributed rate limiting is not implemented

Limits are held in in-memory `FL` and reset on restart; they are per process and not shared across replicas. An attacker can distribute requests across IPs, and multiple instances do not share counters. Put rate limits at a trusted edge and in a shared atomic store (for example Redis), combining IP, account, action, and device signals. Configure proxy trust only for the actual topology. Add tests for multi-instance bypass and registration/recovery abuse.

### High — TURN credentials are static configuration

`/api/rtc/config` returns the configured TURN username and credential to every authenticated caller and has no per-user authorization requirement. If these are long-lived credentials, one user can reuse them until rotated. Use short-lived TURN credentials generated server-side, strict quotas, credential expiry, relay monitoring, and signaling authorization. Verify TURN over TLS where supported. STUN/TURN do not themselves encrypt application signaling; WebRTC DTLS-SRTP protects media transport, not the surrounding account/signaling trust model.

### Medium — backup and persistence model

`db.json` and `db.json.bak` contain the full database, session-token hashes, recovery verifier/vault, and other user metadata. File permissions help on POSIX, but do not encrypt backups at rest or protect them from host/root compromise. The backup copy is a local rotating backup, not a tested disaster-recovery system. Use encrypted off-host backups, least-privilege storage, retention policy, restore drills, and secret/key separation. Review deletion/retention requirements.

### Medium — rate-limit and resource controls are process-local

SSE connection caps and write/signaling counters are in-process. Large JSON objects and media uploads are bounded, but aggregate disk use, connection churn, slow clients, and CPU pressure from synchronous `scryptSync` need load testing and edge controls. Consider async scrypt, global quotas, queue/backpressure handling, and metrics that avoid logging credentials or message contents.

### Medium — server logs and error handling need production review

The server logs startup errors and selected failures, and catches uncaught exceptions without a deliberate graceful-shutdown/health policy. Confirm reverse-proxy/CDN logs never retain query tokens, stream tickets, private request bodies, or personal data. Implement structured redacted logs, alerting, graceful shutdown, and a tested restore/incident procedure.

### Medium — TLS is an operational requirement, not enforced by this Node server

The app can be reached over plain HTTP unless the deployment forces HTTPS. HSTS is emitted only when the incoming request appears HTTPS; forwarded headers are security-sensitive. Enforce HTTPS at the edge, configure exact trusted proxy hops, and test the real deployed headers and origin policy. Do not expose the Node port directly to the public internet.

## Work required before production

1. Commission an independent application-security penetration test with written scope and authorization.
2. Run cross-account authorization tests for every collection and operation, including groups, message mutation/deletion, media ownership, and account deletion.
3. Threat-model and replace the custom static-ECDH messaging scheme with a reviewed protocol implementation that supports authenticated identity keys, device management, forward secrecy and post-compromise security.
4. Test two real accounts across multiple browsers/devices, including account recovery, token theft/revocation, key changes, and offline/online synchronization.
5. Move limits to distributed enforcement; run load and abuse tests.
6. Deploy behind HTTPS, configure short-lived TURN credentials, redact logs, encrypt backups, and prove recovery from backup.
7. Add automated dependency/security scanning and a CI gate for tests.

## Verdict

The existing hardening changes and smoke tests are useful. This review does **not** certify the application as secure, and the current cryptographic/session design has material gaps compared with a mature secure messenger. No production penetration test, real multi-device call test, distributed-load test, backup restore test, or hosting configuration review was performed here.

## Дополнительная проверка v5 (19.10.2026)
- Изменено поведение доверия к ключам: новый ключ не закрепляется автоматически при первой отправке; отправка требует явного действия пользователя после сверки кода безопасности.
- Входящие E2E-сообщения не расшифровываются при отсутствующем/изменившемся закреплённом ключе.
- Добавлен `security-key-verification-test.js`; он проверяет fail-closed проверки клиента. Это регрессионный тест исходного кода, не замена активному penetration test.
- После изменения `npm test` и `node --check` прошли.
- Основной криптографический пробел не устранён: нет проверенного Double Ratchet/forward secrecy, а ключи и клиентский код всё ещё требуют внешнего аудита.

## Re-check after v6 hardening (19.10.2026)
The source now includes a same-origin/explicit-allowlist Origin gate for browser POSTs, an in-memory per-IP POST cap when Redis is absent, image-byte signature validation for uploaded data URLs, generic client-facing responses for unexpected server exceptions, and production rejection of obvious developer-secret placeholders. Regression tests cover the Origin gate and image signature checks. These are code-level checks only; they do not constitute an independent penetration test, production configuration review, or cryptographic audit. The unresolved custom ECDH/no-ratchet limitation remains a blocker for a defensible 7/10 secure-messenger rating.

## Дополнительное усиление v7 (19.10.2026)
- Восстановление аккаунта больше не возвращает фиктивный UID в заметно отличающемся формате: для неизвестного имени выдаются восьмизначный фиктивный UID и соль того же формата. Это уменьшает возможность перечислять существующие аккаунты по одному только ответу `rec/start`; полностью скрыть все побочные каналы без отдельного аудита нельзя.
- При смене облачного пароля сервер ротирует случайный токен сессии и закрывает старые потоковые подключения/билеты. Клиент сохраняет новый токен и переподключается автоматически; украденный прежний токен перестаёт работать.
- Удаление аккаунта теперь требует повторного подтверждения облачным паролем. Одного украденного токена сессии недостаточно; неверное подтверждение блокируется, попытки ограничиваются, а принадлежащие аккаунту медиа удаляются только после успешного подтверждения.
- Добавлены регрессионные проверки формата ответов восстановления, отзыва старого токена при смене пароля, работы нового токена и отказа в удалении аккаунта при неверном подтверждении. `npm test` и проверка синтаксиса встроенного клиентского JavaScript проходят.
- Эти изменения усиливают защиту аккаунта и восстановление доступа, но не заменяют 2FA/passkeys, полноценное управление устройствами, независимый аудит и миграцию E2E на проверенный протокол.

## Дополнение v8 — срок жизни сессий

Срок сессии скользящий: при активности он продлевается, а после года полного бездействия токен отзывается. Выход, смена облачного пароля и восстановление отзывают прежний токен; проверка действует и после перезапуска сервера. Это не заменяет MFA, управление отдельными устройствами или независимый аудит.


## Дополнительное усиление v18 (19.10.2026)

- Проверки облачного пароля при регистрации, восстановлении, смене пароля и удалении аккаунта переведены на асинхронный `crypto.scrypt`, чтобы не блокировать Node.js event loop синхронным вычислением при обработке запроса. Это снижает один из векторов ресурсного истощения, но не отменяет необходимость внешнего rate limiting и нагрузочного тестирования.
- SSE-билеты теперь привязаны к хешу текущей сессии, а endpoint повторно проверяет срок сессии и срок бездействия. Билет до выхода не открывает поток после отзыва токена.
- Проверено: `node --check server.js` и полный `npm test` проходят. Это регрессионные тесты в локальной среде, не независимый аудит и не нагрузочный тест.
