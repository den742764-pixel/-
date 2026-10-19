# Sirius v25 — release notes

- New accounts receive exactly 50 Sirius Stars once at registration; the server records the welcome bonus in the audit log. Existing accounts are not retroactively credited.
- Stars remain purchasable through the existing order/manual-confirmation flow. No task/achievement reward path was added.
- Profile photo picker from device gallery retained and covered by a source-level regression check.
- Added 12 preset avatar choices.
- Added 8 preset profile cover designs.
- Added a gallery-based chat wallpaper picker and a control to remove it.
- Expanded Sirius Premium benefits shown in the server state: profile glow/frame, exclusive covers and avatars, expanded themes, extra reactions, priority support, early access, and Premium identity markers.
- Added regression coverage for gallery pickers, presets, welcome bonus, star-credit behavior and Premium flows.

## Verification

`npm test` passes all security, crypto, runtime, manual payment confirmation, Premium bot and profile-media regression tests in this build. The browser file pickers are validated by integration/source tests; physical-device selection should still be smoke-tested on the deployment device.
