# Sirius cryptography status — release gate

## Current implementation (do not market as Signal-equivalent)
- Browser WebCrypto uses ECDH P-256 + HKDF-SHA-256 + AES-256-GCM for direct-message payloads.
- New direct messages use envelope version 2: a fresh random 128-bit HKDF salt and 96-bit GCM nonce per message, plus authenticated associated data binding the ciphertext to conversation ID, sender, recipient, message ID, and timestamp. Client regression tests verify valid round-trip decryption and reject ciphertext tampering and metadata substitution.
- Legacy version-1 direct messages remain readable for compatibility. New sends use version 2.
- Password-derived vault key uses PBKDF2-HMAC-SHA-256 with 250,000 iterations. The server stores the encrypted private-key vault and a derived verifier, not the password itself.
- The client persists a session bundle in IndexedDB encrypted with AES-GCM using a non-extractable browser CryptoKey; the bundle contains the token and an exportable private-key JWK. This is stronger than plaintext localStorage but does not protect against active same-origin XSS, malicious extensions, a compromised browser/device, or an attacker able to invoke the key in the same origin.
- The server validates public identity keys as P-256 EC JWKs. The client pins each peer key locally after first contact and blocks sending if a previously pinned key changes. Users still must compare the safety code out-of-band: first-contact key substitution is not prevented by local pinning, and local pins can be lost or modified with browser storage.
- The direct-message key agreement still uses long-lived identity private keys. There is no reviewed Double Ratchet, per-message forward secrecy, post-compromise security, signed/one-time pre-key bundle protocol, complete multi-device protocol, or independent key-transparency service.
- Group chats, call signaling, profile/social data, and some media/social features are not guaranteed to be end-to-end encrypted by the direct-message implementation.

## Release gate for an honest 9/10 cryptography target
1. Integrate a maintained, independently reviewed Signal Protocol implementation compatible with the supported clients; do not hand-roll Double Ratchet.
2. Add identity keys, signed prekeys, one-time prekeys, authenticated device bundles, replay protection, ratchet state persistence, and key-change warnings.
3. Define a threat model and protocol versioning; authenticate all message metadata as AEAD associated data.
4. Encrypt attachments client-side with fresh random keys; deliver keys only inside E2EE envelopes. Ensure thumbnails, previews, and backups follow the same model.
5. Design secure multi-device linking, device revocation, and backup/recovery separately; explain tradeoffs to users.
6. Add protocol test vectors, negative tests, fuzzing, cross-client interoperability tests, dependency scanning, and an external cryptographic audit before claiming 9/10.

## Security boundary
TLS protects transport, not a compromised device or server metadata. End-to-end encryption cannot hide all metadata (such as timing and traffic volume). This document is a release checklist, not proof of implementation or certification.

## Изменения v5 — обязательная проверка ключа
- Для личных E2E-сообщений первая отправка блокируется, пока пользователь явно не подтвердит закрепление ключа после сверки кода безопасности.
- Входящие E2E-сообщения скрываются, если ключ собеседника не закреплён или изменился; приложение не пытается расшифровать их молча под новым ключом.
- После подтверждения нового ключа поток переподключается, чтобы заново обработать ожидающие сообщения.
- Это уменьшает риск незаметной подмены ключа, но не добавляет ratchet, не обеспечивает прямую секретность и не заменяет независимую проверку кода.

## Дополнительное усиление v7 (19.10.2026)
- Восстановление аккаунта больше не возвращает фиктивный UID в заметно отличающемся формате: для неизвестного имени выдаются восьмизначный фиктивный UID и соль того же формата. Это уменьшает возможность перечислять существующие аккаунты по одному только ответу `rec/start`; полностью скрыть все побочные каналы без отдельного аудита нельзя.
- При смене облачного пароля сервер ротирует случайный токен сессии и закрывает старые потоковые подключения/билеты. Клиент сохраняет новый токен и переподключается автоматически; украденный прежний токен перестаёт работать.
- Удаление аккаунта теперь требует повторного подтверждения облачным паролем. Одного украденного токена сессии недостаточно; неверное подтверждение блокируется, попытки ограничиваются, а принадлежащие аккаунту медиа удаляются только после успешного подтверждения.
- Добавлены регрессионные проверки формата ответов восстановления, отзыва старого токена при смене пароля, работы нового токена и отказа в удалении аккаунта при неверном подтверждении. `npm test` и проверка синтаксиса встроенного клиентского JavaScript проходят.
- Эти изменения усиливают защиту аккаунта и восстановление доступа, но не заменяют 2FA/passkeys, полноценное управление устройствами, независимый аудит и миграцию E2E на проверенный протокол.

## Дополнение v8 — срок жизни сессий

Срок сессии скользящий: при активности он продлевается, а после года полного бездействия токен отзывается. Выход, смена облачного пароля и восстановление отзывают прежний токен; проверка действует и после перезапуска сервера. Это не заменяет MFA, управление отдельными устройствами или независимый аудит.

## v16 server-side enforcement
- New direct messages are rejected unless their payload kind is `e2e` and their envelope version is supported; the server does not trust an `e` flag alone.
- This is a format/policy guard, not proof that all clients encrypt correctly or that group messages and attachments are end-to-end encrypted.


## Статус v17 / 1.10.0

В v17 не менялся формат E2E-шифротекста. Добавлены исправления приватности профиля и серверные лимиты сообщений. Ограничения E2E, описанные выше, остаются в силе.
