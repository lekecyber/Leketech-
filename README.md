# Leketech referral app starter

This is a **starter project**, not a live payment service. It implements account registration/login, referral links, a KSh 200 Daraja STK request, payment callback handling, and a KSh 50 wallet credit for one verified successful registration per referral. It does **not** send real automatic payouts yet.

## Current configuration

- App name: Leketech
- Registration fee: KSh 200
- Till configured by default: `6285683`
- Referral reward: KSh 50 after a referred member's payment is confirmed
- Payment method: M-Pesa Daraja STK Push
- Payout: disabled until the separate B2C payout integration is approved and completed

## Requirements

- Node.js 20+
- A public HTTPS domain for the callback
- Safaricom Daraja app credentials and a shortcode/Till configuration approved for the API products you use
- For automatic payouts: separate production B2C approval and credentials

## Run locally

1. Install Node.js 20 or newer.
2. Extract this ZIP and open a terminal in the `leketech_app` directory.
3. Run `npm install`.
4. Copy `.env.example` to `.env`.
5. Set `JWT_SECRET` to a long random secret (at least 32 characters), and change `ADMIN_PASSWORD`.
6. Keep `DARAJA_ENV=sandbox` during development. Add sandbox Consumer Key, Consumer Secret, Passkey and test shortcode from your Daraja dashboard.
7. Set `BASE_URL` and `MPESA_CALLBACK_URL` to your public HTTPS callback address when testing callbacks. A local server alone cannot receive Safaricom callbacks.
8. Run `npm start`, then open `http://localhost:3000`.

## Deploy (Render, Railway, or a VPS)

1. Create a Node.js web service and upload/push this project.
2. Set the start command to `npm start` and Node version to 20+.
3. Add every needed environment variable from `.env.example` in the host's secret/environment settings. **Do not commit `.env` or put secrets in frontend code.**
4. Attach persistent storage for the `data/` directory. SQLite data can be lost on hosts with ephemeral disks; for production, use a managed PostgreSQL database and migrate the schema before launch.
5. Add a domain with HTTPS and set `BASE_URL` plus `MPESA_CALLBACK_URL` to that domain.
6. Test registration and callbacks in Daraja sandbox before applying for production access.
7. Confirm the shortcode/Till transaction type is supported for your account. This project uses `CustomerBuyGoodsOnline`; Safaricom may require different business shortcode/Head Office configuration for a particular Till.
8. Only switch `DARAJA_ENV=production` after Safaricom grants production access and you have verified the live configuration.

## M-Pesa flow

The browser requests an STK push from the backend. The backend calls Daraja; Safaricom sends the final result to the public callback. The callback must confirm `ResultCode === 0`, a receipt, and the transaction amount before the app activates the member and records a reward. A successful initial API response means only that the prompt was accepted, not that payment completed.

## Automatic payouts are not yet active

Do not describe payouts as automatic or promise withdrawals yet. Automatic M-Pesa payments to members require a separate approved B2C integration, secure credential storage, result/timeout callbacks, idempotent payout ledger, transaction reconciliation, limits, fraud review and sufficient funds. The payout endpoint intentionally refuses requests until this is implemented and reviewed.

## Production checklist — required before public launch

- Replace SQLite with managed PostgreSQL and use database migrations.
- Use HTTPS, secure cookies or short-lived tokens, strict rate limits, monitoring and backups.
- Add email/phone verification, password reset, admin MFA and audit logs.
- Verify callbacks against Daraja transaction status where possible, reconcile missed callbacks, make all webhook processing idempotent, and test duplicate/out-of-order callbacks.
- Implement a transaction ledger and payout state machine; never let a browser mark payments successful or edit balances.
- Add terms of service, privacy notice, refunds, support contact and clear referral eligibility rules.
- Have Kenyan counsel review the referral model, consumer-protection, tax, data-protection and any applicable direct-selling/financial-services requirements. Ensure rewards are funded sustainably and are not represented as guaranteed income.
- Conduct a security review and test with low-value sandbox transactions first.

## Important limitation

This code is a functional development baseline, not a certified or production-audited financial application. The Till number alone does not grant API access. Never share your M-Pesa PIN, Daraja Consumer Secret or Passkey in chat.


## Quick website deployment with Render

The repository includes `render.yaml` as a starting point for a Render web service.

1. Create a GitHub repository and upload the extracted project files (do not upload `.env`, `data/`, or secrets).
2. In Render, choose **New + → Blueprint** and connect the repository containing `render.yaml`, or create a Node web service manually.
3. Check the persistent disk is mounted at `/opt/render/project/src/data` if using the supplied Render configuration.
4. Set `ADMIN_EMAIL` and a strong `ADMIN_PASSWORD`. Set `BASE_URL` to the HTTPS URL Render gives you.
5. Set `MPESA_CALLBACK_URL` to `https://YOUR-SITE/api/mpesa/callback/YOUR-LONG-RANDOM-TOKEN`; use that same URL in the `MPESA_CALLBACK_URL` environment variable. The server checks the secret final path segment.
6. Add Daraja sandbox credentials and test before requesting production access.
7. After deployment, open `https://YOUR-SITE/api/config` to confirm the service responds, then open the base website URL to see the Leketech interface.

Do not treat a generated hosting URL as proof that M-Pesa is connected. Payment callbacks need to be reachable over HTTPS, and actual STK requests need valid Daraja credentials and the correct approved shortcode/transaction type. For public production use, migrate to managed PostgreSQL and finish the separate B2C payout integration.
