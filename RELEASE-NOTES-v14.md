# Sirius Security v14 (1.7.0)

## Message input validation

- Added server-side validation of message records before persistence or broadcast.
- Rejects malformed conversation IDs, missing/unknown senders, non-finite/negative timestamps and timestamps more than five minutes in the future.
- Bounds message body length to 500,000 characters, kind length to 32, gift-number length to 80, receipt-map keys to 500, and each receipt array to 500 valid user IDs.
- Restricts envelope version to the supported values 1 and 2 and validates read-state shape.
- Added regression tests for far-future timestamps and oversized receipt metadata.

## Compatibility and limits

- Existing supported message fields and the current E2E v2 envelope remain unchanged.
- This is server-side input hardening, not a replacement for a reviewed end-to-end encryption protocol or an independent security audit.
- Session inactivity expiry remains one year; activity extends the inactivity window.
