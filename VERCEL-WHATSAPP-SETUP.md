# TOMZY ARCHITECTURAL — Vercel + WhatsApp AI backend

This version adds the server-side pieces needed to connect the existing website AI to WhatsApp Cloud API while keeping secrets off the browser.

## API routes

- `POST /api/chat` — website AI chat; creates/continues a lead conversation.
- `POST /api/leads` — website Request a Quote intake; stores the lead and can notify the admin on WhatsApp.
- `GET/POST /api/whatsapp/webhook` — Meta webhook verification and incoming WhatsApp messages.
- `GET /api/health` — backend configuration health check (does not expose secret values).

## Environment variables in Vercel

Add these under **Project → Settings → Environment Variables**. Keep them server-side and do not commit a real `.env` file.

Required for website AI:
- `OPENAI_API_KEY`
- `OPENAI_MODEL` (use a model available to your OpenAI API account)

Required for WhatsApp:
- `WHATSAPP_API_VERSION` (the Graph API version you select in Meta)
- `WHATSAPP_PHONE_NUMBER_ID`
- `WHATSAPP_ACCESS_TOKEN`
- `WHATSAPP_VERIFY_TOKEN` (a secret string you choose for webhook verification; this is NOT the Meta access token)
- `META_APP_SECRET`

Recommended for persistent shared lead/conversation records:
- `DATABASE_URL` for a Postgres/Neon database

Optional:
- `ADMIN_WHATSAPP_TO` — WhatsApp destination for quote/negotiation/review alerts. Use the international format expected by the Cloud API, without the `+` sign unless Meta's current API instructions for your setup say otherwise.

## Database

`db/schema.sql` contains the tables for leads and messages. The backend also creates them automatically when `DATABASE_URL` is present.

The same lead record can be used by the website and WhatsApp:
- Website browser stores a `tomzyLeadId` locally.
- WhatsApp conversations use `wa_<phone_number>` as the lead ID.
- Messages are stored with their channel (`website` or `whatsapp`).

## Meta webhook setup

After the site is deployed, the callback URL is:

`https://YOUR-VERCEL-DOMAIN/api/whatsapp/webhook`

Use the exact same `WHATSAPP_VERIFY_TOKEN` value in Meta's Verify token field.

Do not paste the WhatsApp access token or Meta app secret into the website code or send them in chat screenshots.

## Important production note

The backend is prepared for WhatsApp Cloud API, but the real Meta account still needs its production phone number, webhook subscription, production access token and any required business/phone setup. The current Meta test token should not be treated as the final production credential.
