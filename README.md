# Your Gallery

**Your Gallery — Images worth keeping.** A Cloudflare Workers image marketplace starter built with plain HTML, CSS, vanilla JavaScript, Workers modules, D1, and private R2 storage.

> **Deployment status:** This repository is source code only. It has not been deployed to a Cloudflare account and no live Khalti transaction has been run. Do not enable live payments until the production checklist below is completed.

## Included

- Responsive editorial-style storefront, categories, gallery search/filter/sorting, image detail, creator upload form, account forms, library, contact and copyright-report forms.
- Worker API with D1-backed registration/login/session records, PBKDF2 password hashes, HttpOnly cookies, same-origin checks for mutations, creator ownership checks, image moderation, basic admin overview/user/order/report/settings endpoints, audit records, and secure download authorization.
- Private R2 original objects and separate preview object keys; original downloads stream through the Worker after a free-license or purchase authorization check. Public image previews intentionally show a placeholder until a safe server-side processing pipeline is configured, so full-resolution uploads are not exposed as previews.
- Khalti ePayment initiation and server-to-server lookup verification using the current ePayment v2 routes: `/api/v2/epayment/initiate/` and `/api/v2/epayment/lookup/`.
- D1 schema migration and a small automated security utility test suite.

## Requirements

- Node.js 20 or newer recommended
- A Cloudflare account with Workers, D1, and R2 enabled
- A Khalti merchant account and test credentials before payment testing

## Configure Cloudflare

1. Install dependencies: `npm install`.
2. Create D1: `npx wrangler d1 create your-gallery-db`.
3. Paste the returned database ID into `wrangler.toml` where `REPLACE_WITH_D1_DATABASE_ID` appears.
4. Create the R2 bucket: `npx wrangler r2 bucket create your-gallery-images`.
5. Copy `.dev.vars.example` to `.dev.vars`; set a long random `SESSION_SECRET`, `ADMIN_EMAIL`, and a strong one-time `ADMIN_PASSWORD`. Set Khalti **test** secret/public keys from your own merchant dashboard.
6. For local development, set `SITE_URL = "http://localhost:8787"` in `wrangler.toml` and `KHALTI_BASE_URL = "https://dev.khalti.com"`.
7. Apply schema: `npm run db:local`.
8. Run: `npm run dev` and open the URL printed by Wrangler.

The first API request bootstraps the configured admin account only if that email does not already exist. After initial setup, remove `ADMIN_PASSWORD` from Worker secrets/vars and rotate any temporary password using a planned account-management flow; this starter does not include password reset/change endpoints.

## Deploy

1. Set `SITE_URL` to the final HTTPS origin and keep `KHALTI_BASE_URL = "https://dev.khalti.com"` for test mode.
2. Apply migrations remotely: `npm run db:remote`.
3. Set secrets using Wrangler rather than committing them: `npx wrangler secret put KHALTI_SECRET_KEY`; `npx wrangler secret put SESSION_SECRET`; configure `ADMIN_EMAIL` and a strong `ADMIN_PASSWORD` securely in your deployment workflow for the first admin bootstrap. Do not commit `.dev.vars`.
4. Deploy: `npm run deploy`.
5. Verify `/api/health`, account creation/login/logout, R2 upload, moderation, library/download authorization, and a complete Khalti test payment before switching to live keys.
6. For production, change `KHALTI_BASE_URL` to `https://khalti.com` only after merchant approval and successful sandbox tests. Follow the latest official Khalti docs: https://docs.khalti.com/khalti-epayment/ and https://docs.khalti.com/api/transaction_status/.

## API overview

| Area | Endpoint(s) |
|---|---|
| Health / categories | `GET /api/health`, `GET /api/categories` |
| Authentication | `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me` |
| Public images | `GET /api/images`, `GET /api/images/:id`, `GET /media/:id` |
| Creator | `POST /api/images`, `GET /api/creator/images`, `GET /api/creator/earnings` |
| Checkout | `POST /api/checkout`, `POST /api/payments/khalti/initiate`, `POST /api/payments/khalti/verify` |
| Orders / downloads | `GET /api/orders`, `GET /api/library`, `GET /api/download/:imageId` |
| Contact / copyright | `POST /api/contact`, `POST /api/reports` |
| Admin | `GET /api/admin/stats`, `GET /api/admin/images`, `POST /api/admin/images/:id/moderate`, `GET /api/admin/users`, `PATCH /api/admin/users/:id/role`, `PATCH /api/admin/users/:id/status`, `GET /api/admin/orders`, `GET /api/admin/reports`, `GET/PATCH /api/admin/settings` |

## Important production gaps — do not skip

This is a functional starting implementation, not a certified production marketplace. Complete the following before accepting real money or untrusted public uploads:

1. **Image processing (blocking):** Public gallery cards currently use a placeholder because no server-side image processor is configured. A separate R2 preview object is kept private, and the `/media/:id` route deliberately refuses to expose it. Configure and test a server-side image-processing pipeline (for example Cloudflare Images or a dedicated image processor) to create resized, watermarked, EXIF-scrubbed public previews. Do not expose preview objects or launch paid listings with public previews until this is complete.
2. **Abuse controls:** Add distributed rate limiting for login/register/upload/contact/report, bot protection, quotas, and alerting. A per-process memory limiter is not sufficient for Workers.
3. **Security review:** Add explicit CSRF tokens for cookie-authenticated mutations, email verification, password reset/change, session rotation/revocation on credential changes, account deletion/export, security headers/CSP, and independent penetration testing. Same-origin checks are only a layer, not a replacement for CSRF tokens.
4. **Payment hardening:** Validate every Khalti response field required by the merchant agreement, order identity and currency semantics, concurrent/idempotent finalization under retries, refund webhooks/workflow, and reconciliation. Test all failure/cancel/pending/duplicate scenarios with Khalti sandbox. The app intentionally does not fake successful payments.
5. **Business operations:** Creator payout/settlement is not implemented. Platform fee is stored as a setting but not yet applied to creator settlement. Refund operations are not integrated with Khalti. Free download policy currently allows free original downloads only for `price_npr=0` and `license_type='free'`.
6. **Admin UX:** Core moderation and account controls are implemented; richer filtering, pagination, report status updates, detailed audit-log viewing, content/collection management, and image-processing review are still needed.
7. **Privacy/legal:** Policy pages are starter drafts, not legal advice. Review with qualified counsel and fill in operator identity, contact details, retention schedules, governing law, refund rules, and all required disclosures.
8. **Accessibility/QA:** Run keyboard, screen-reader, contrast, responsive, browser, load, and end-to-end tests. The image detail page is a separate route; payment flow needs end-to-end verification in deployed sandbox conditions.
9. **Operations:** Configure backups/restore drills for D1, R2 lifecycle and deletion jobs, monitoring, logs without secrets, incident response, and production environment separation.

## Testing

Run `npm test` for the included password-hashing/session utility tests. Run `npx wrangler deploy --dry-run` to validate the Worker bundle once your Cloudflare configuration is complete. A successful local unit test does not mean that R2, D1, or Khalti has been end-to-end tested.

## Sample photography

The hero and collection artwork reference remote Unsplash image URLs for layout illustration only. Verify each image's license and production availability before relying on these images. They are not represented as photography created by the platform owner.
