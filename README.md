# GLOVAL NIG LTD Marketplace

A Django/PostgreSQL starter marketplace for clients and freelancers, with role-specific dashboards, project listings/proposals, project file uploads, conversations, NGN/USD wallet ledgers and Paystack NGN deposit initialization/verification.

## Included
- Custom user model with client/freelancer/admin roles.
- Service categories, project posts, freelancer proposals and assignment.
- Project workflow: open → assigned → in progress → review → completed.
- Project file uploads with extension and size allow-list checks.
- Job-scoped conversations and message attachments.
- NGN and USD wallet records with immutable ledger entries.
- Paystack NGN transaction initialization, verification and signed webhook handling.
- Admin site for operational records.
- Responsive navy/teal frontend and separate role dashboards.

## Important implementation boundaries
This is a runnable MVP foundation, not a production-certified escrow/payment institution. Before accepting real funds:
1. Obtain legal, tax and payment-provider approval for the marketplace/funds-flow model.
2. Configure Paystack live credentials and webhook URL. Paystack checkout is implemented for NGN only. USD deposits deliberately fail until an eligible international processor is integrated and tested for your registered business country.
3. Configure a real payout workflow, beneficiary verification, transfer webhooks, withdrawal approvals, reconciliation and refund/dispute procedures. The included payout and withdrawal models are foundations; automated withdrawals are not enabled.
4. Configure private object storage and authenticated download delivery for project/message files. Do not expose `MEDIA_ROOT` publicly in production. The local development server is not suitable for production file delivery.
5. Add email verification, password reset email delivery, rate limiting, CAPTCHA, audit logging, malware scanning, content-disposition downloads, antivirus scanning and a documented privacy/retention policy.
6. Review all permissions and transaction flows with a security engineer. Use HTTPS and secure secrets management.

Wallet entries are immutable application records. They are not a substitute for a regulated client-money/escrow arrangement or a bank account balance. Currency balances are separate; do not silently convert NGN and USD. If FX is introduced, store the quote, source, timestamp, fees and both original and converted amounts.

## Local setup (Windows PowerShell)

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
```

Edit `.env`. For local development you can keep SQLite by removing/commenting `DATABASE_URL`; PostgreSQL is recommended for real development and deployment.

Create PostgreSQL database `gloval_db`, then set `DATABASE_URL` in `.env`, for example:

```env
DATABASE_URL=postgresql://postgres:YOUR_PASSWORD@localhost:5432/gloval_db
```

Run:

```powershell
python manage.py makemigrations marketplace
python manage.py migrate
python manage.py createsuperuser
python manage.py seed_services
python manage.py runserver
```

Open http://127.0.0.1:8000/ and `/admin/`.

## Paystack setup
- Use test keys first: `PAYSTACK_SECRET_KEY` and `PAYSTACK_PUBLIC_KEY` in `.env`.
- Set `SITE_URL` to the externally reachable HTTPS site URL.
- Register webhook endpoint: `https://YOUR_DOMAIN/payments/paystack/webhook/` in Paystack dashboard.
- Never put the secret key in JavaScript or templates.
- Server verifies reference, transaction status, amount and currency before crediting wallet. Webhook signature uses HMAC SHA-512 and successful deposit is idempotent by unique ledger reference.
- Paystack NGN amounts are converted to kobo (`amount * 100`).

## Default services
The seed command adds Graphic Design, Web Design & Development, Technical Support, Customer Service, Book Publishing, Writing, Application Development, Software Development and Digital Consultancy.

## Roles
Users can self-register only as `client` or `freelancer`. Admin privileges must be assigned by a trusted superuser in Django Admin; never allow public registration as admin.

## Production checklist
- Set `DEBUG=False`, a strong random `SECRET_KEY`, correct `ALLOWED_HOSTS` and secure `DATABASE_URL`.
- Set `CSRF_TRUSTED_ORIGINS`, email provider and secure storage settings as needed.
- Run `python manage.py check --deploy` and `python manage.py collectstatic`.
- Use PostgreSQL backups, database TLS, private file storage, malware scanning, monitoring and tested restore procedures.
- Configure a production WSGI/ASGI server and reverse proxy with HTTPS.
