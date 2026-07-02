# Never Miss a Lead (LITE) — free n8n missed-call text-back

I run a diesel repair business. When I'm under a truck, I can't answer the phone — and every call that rings out is a customer dialing the next shop. A 2016 study by 411 Locals (85 businesses, 58 industries, 30 days of monitored calls) found only **37.8% of calls to small businesses get answered live**. The rest hit voicemail or nothing.

So I built the boring fix in n8n for my own shop: **missed call → instant text-back → lead logged.** This repo is that core, free, MIT-licensed, no email gate.

```
Webhook → Normalize Lead → SMS Text-Back (Twilio) → Log to Google Sheets → Respond
```

## What the LITE version does

- Catches a **missed call** (via Twilio) or a **web form POST** on one webhook
- Instantly texts the caller back from your business number (message is yours to edit — the default is under 160 chars and single-segment safe)
- Logs every lead to a Google Sheet (`Timestamp | Name | Phone | Message | Source`)
- Returns `{ success: true }` so form integrations don't hang

## Setup (~20 minutes)

1. **Import** `never-miss-a-lead-lite.workflow.json` into n8n (Cloud or self-hosted): **... menu → Import from File**.
2. **Credentials:** create your own Twilio + Google Sheets credentials in n8n and attach them. The `REPLACE_*` ids in the JSON are placeholders — n8n never exports real credentials, so there are no secrets in this file and yours must be created fresh.
3. **Personalize:** in the `SMS Text-Back` node, replace `{{BUSINESS_NAME}}` and set `from` to your Twilio number (the `+1469555...` numbers in the JSON are fictional placeholders). In `Log Lead`, pick your spreadsheet — tab named `Leads`, header row `Timestamp | Name | Phone | Message | Source`.
4. **Missed-call wiring (Twilio):** a 4-widget Twilio Studio flow on your number — *Incoming Call → Connect Call To (your real phone) → Split Based On → HTTP Request*. Hook the Split to the Connect widget's **Connected Call Ended** transition (Studio has no "no answer" transition — the outcome lives in a variable), test `{{widgets.YOUR_CONNECT_WIDGET.DialCallStatus}}` with one condition **Matches Any Of** `no-answer, busy, failed`, and point that condition at an HTTP Request widget: POST, Form URL Encoded, to this workflow's production webhook URL. In the HTTP Request widget add **two HTTP parameters yourself** — `From` = `{{trigger.call.From}}` and `CallStatus` = `{{widgets.YOUR_CONNECT_WIDGET.DialCallStatus}}` — the widget only sends parameters you add; the caller's number is NOT included automatically. The workflow reads `From`/`CallStatus` and the missed call becomes a logged lead. Web forms can POST JSON `{ name, phone, message, source }` to the same webhook.
5. **Test:** `curl -X POST <test-url> -H "Content-Type: application/json" -d '{"name":"Test","phone":"+15555550123","message":"hello","source":"website"}'` → node lights, text received, sheet row appended. Then **Activate**.

## ⚠️ Before real US traffic: A2P 10DLC

US carriers filter or block business SMS from unregistered 10-digit numbers. Register a Brand + Campaign in the Twilio Console (small carrier fees, a few days to approve) or your texts will silently not arrive. Only text people who just contacted you, and leave Twilio's STOP/opt-out handling on. Not legal advice — read Twilio's A2P 10DLC docs.

## LITE vs. the full version

This LITE core is genuinely useful and yours forever. The paid version is the full system I built for my own shop, plus the sales-side assets:

| | LITE (this repo) | Full |
|---|---|---|
| Instant text-back + Sheets log | ✅ | ✅ |
| Missed-call (Twilio `From`/`CallStatus`) handling | ✅ | ✅ |
| **Urgency routing** (Urgent / Quote / Fallback → 3 different replies) | — | ✅ |
| **Email auto-confirmation** to the lead | — | ✅ |
| **Owner SMS alert** with the full lead ticket | — | ✅ |
| **SMS copy pack** — tested reply templates, all <160 chars / GSM-7 safe | — | ✅ |
| **ROI worksheet** — the missed-call math, honestly sourced | — | ✅ |
| Setup guide (MD + PDF) with A2P 10DLC walkthrough + per-trade keyword tables + troubleshooting | — | ✅ |
| Agency license option — deploy for unlimited clients, keep 100% of your fees | — | ✅ |

**Full version:** $47 single-business / $97 agency-resell →
- Payhip (instant download): `[PAYHIP-PRODUCT-URL]`
- Gumroad: `[GUMROAD-PRODUCT-URL]`

*(Agencies: missed-call text-back is a flagship feature of GoHighLevel's $297/mo Unlimited plan (gohighlevel.com/pricing, retrieved 2026-07-02). This is the self-hosted version of that offer — you sell the outcome and keep the margin.)*

## License

LITE version: [MIT](LICENSE) — use it, modify it, deploy it for clients, no strings.

Built by a mechanic who got tired of losing jobs to voicemail.
