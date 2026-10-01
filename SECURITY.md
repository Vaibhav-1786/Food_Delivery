# Security Policy

Quickbite handles user accounts, wallet balances and payment flows. Security reports are taken seriously.

## Supported versions

Only the latest commit on the `main` branch receives security fixes.

## Reporting a vulnerability

**Please do not open a public issue for security problems.**

1. Use GitHub's private reporting: **Security → Report a vulnerability** on this repository (preferred), or
2. Email the maintainer, Vaibhav Chauhan, at "vaibhavchauhan1786@gmail.com".

Please include a description, impact, steps to reproduce (endpoint, role, request/response) and a suggested fix if you have one. You can expect an acknowledgement within **7 days** and a status update within **14 days**. Please allow reasonable time for a fix before public disclosure.

### In scope
Role bypass between customer / restaurant / admin, cross-restaurant data access, wallet or GK-reward manipulation, Razorpay signature-verification bypass, OTP / password-reset flaws, JWT issues, injection, secrets exposure.

### Out of scope
Issues that only exist when development defaults are left in production (see checklist), denial of service by volume, missing rate limiting on its own, social engineering, and flaws in third-party services (Razorpay, Google, OpenRouter, SMTP providers).

## Security measures in the project

- Passwords hashed with bcrypt; password hashes are never returned by the API.
- JWT tokens carry the role; sensitive routes are restricted with `@token_required([...])` on the backend.
- Wallet debits, GK rewards, Razorpay signature checks and fraud checks run server-side only.
- GK rewards can pay out once per session (unique constraint on `game_rewards.session_id`).
- Email OTPs are stored only as SHA-256 hashes, are single-use, purpose-scoped, expire (default 5 min), limit attempts (default 5) and enforce a resend cooldown.
- Password reset uses short-lived, single-use hashed tokens; forgot-password responses are generic to avoid account enumeration.
- Restaurant resources are checked for ownership before edit (cross-restaurant access returns 404).
- Fraud-detection service and admin fraud center flag suspicious activity.

## Production deployment checklist

The repository ships with **development defaults**. Before going live:

- [ ] Set long random values for `SECRET_KEY` and `JWT_SECRET_KEY`.
- [ ] Change or delete the seeded accounts (`admin@fooddelivery.com / Admin@123`, and the four `Restaurant@123` restaurants).
- [ ] Set `OTP_DEBUG_MODE=0` (when on, mobile OTP codes are returned in API responses) and wire a real SMS provider.
- [ ] Set `EMAIL_ENABLED=1` with working SMTP credentials.
- [ ] Configure `DB_HOST` / `DB_USER` — without them the app silently falls back to a local SQLite file, which is for quick tests only.
- [ ] Use a dedicated MySQL user with least privilege, not `root`.
- [ ] Do not run the Flask development server; use gunicorn (or similar) behind a reverse proxy with HTTPS.
- [ ] Set `FRONTEND_URL` and CORS settings to your real domain.
- [ ] Never commit `backend/.env`; keep `MAIL_*`, `RAZORPAY_KEY_SECRET` and `AI_API_KEY` backend-only (never in `VITE_*` variables).
- [ ] Use live Razorpay keys only over HTTPS and verify webhooks/signatures in production.
- [ ] Add rate limiting on login, OTP and password-reset endpoints.

## Known limitations

- No automated test suite yet.
- Mobile OTP has no real SMS gateway wired in by default.
- Food and restaurant images are URL-based (no upload validation surface yet).
- The AI assistant sends user prompts to a third-party provider (OpenRouter).
