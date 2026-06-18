# MVP Build Plan — accountant-app

This turns the research/strategy and the clickable prototype (`prototype/index.html`) into a
concrete, executable engineering plan. It is the bridge between "what good looks like" and the
first shippable product.

**Product framing (locked):**
- **White-label per firm**, **multi-tenant** backend, **single shared app themed at runtime** (no per-firm store binaries — avoids Apple Guideline 4.3 cloned-app rejections).
- **Standalone first** — capture + engagement value with no accounting-platform dependency; Xero/MYOB sync is a fast-follow.
- **React Native + Expo**, single codebase iOS/Android, Expo push.
- **Flat per-practice pricing** (deliberately anti-Dext-per-client).
- **Adoption-first**: win the first 90 seconds (passwordless), be genuinely mobile-first, ≤3 taps to capture, replace WhatsApp/email with compliant in-app messaging.

---

## B1. Repo / project structure

Because the project lives in a single repo the owner controls, organize it as a **mono-repo**:

```
accountant-app/
├── prototype/          # current clickable HTML mockup (design reference)
├── docs/               # this plan + future ADRs
├── app/                # React Native + Expo client app (iOS/Android, Expo web)
├── console/            # Firm admin console (web — React/Next.js)
├── backend/            # Multi-tenant API + workers (Node/TypeScript)
└── packages/
    ├── shared/         # shared TS types, validation (zod), tax constants (rates by FY)
    └── ui/             # shared theme tokens (--brand mapping) + primitives
```

`app` and `console` both consume `packages/ui` so the white-label theme tokens
(`--brand`, `--brand-ink`, `--brand-soft`) are defined once and applied at runtime per tenant.
(If split repos are ever preferred, the service boundaries below are unchanged.)

---

## B2. Tech stack

**Mobile app (`app/`)**
- Expo (managed) + **Expo Router**; TypeScript.
- `expo-camera` for capture; `expo-image-manipulator` for downscale/compress before upload.
- `expo-local-authentication` for biometric unlock (Face ID / fingerprint).
- **Offline-first**: `expo-sqlite` (or WatermelonDB) local store + a sync queue, so capture/log work with no connectivity and reconcile later (the ATO myDeductions baseline is offline; we must match it).
- Push: **Expo Notifications** for delivery + **OneSignal** for segmentation/campaigns.
- State/data: TanStack Query over a typed API client; forms with react-hook-form + zod.

**Firm console (`console/`)**
- React + Next.js (web), same design tokens; server-side calls to the same API.

**Backend (`backend/`)**
- Node/TypeScript — **NestJS** (or Fastify) modular monolith to start.
- **Postgres** with **row-level multi-tenancy**: every tenant-scoped table carries `tenant_id`; enforce with Postgres RLS policies keyed off the authenticated tenant claim.
- **Redis + BullMQ** for async jobs: OCR processing, scheduled/segmented push, newsletter generation, export-bundle building.
- **S3-compatible object storage in an AU region** (receipt images/PDFs, export zips); pre-signed URLs for up/download; lifecycle policy guaranteeing ≥5-year retention.
- Local dev: `docker compose` (Postgres + Redis + MinIO).

**Auth**
- **Passwordless**: email magic-link + SMS OTP (no passwords — the #1 adoption failure point).
- Long-lived refresh token + short-lived access JWT carrying `tenant_id` + `role`.
- Device **biometric unlock** gates app re-entry; framed as "save your face for next time" (the prototype's `ob-faceid` screen).

**OCR / AI**
- **Taggun** as primary receipt OCR (AU-friendly, extracts ABN/GST cheaply).
- **Claude API (vision)** fallback for messy/handwritten receipts, plus the conversational logging and EOFY-summary features.
- Contractual: **no-training / no-retention**, AU data residency, and **TFN-class data never sent** to any external model.
- Always return a **confidence score** and route to human-in-the-loop confirm (the prototype's AI suggestion card on `capture`).

**Billing / provisioning**
- **Stripe** subscriptions; a webhook provisions the tenant and unlocks console access (self-serve, no manual onboarding step).

---

## B3. Core data model

All tenant-scoped tables carry `tenant_id`. Each entity maps to the prototype screen(s) that
consume it:

| Entity | Key fields | Drives screen(s) |
|---|---|---|
| `Tenant` (firm) | name, logo URL, `brand_color`/`brand_ink`/`brand_soft`, ABN, plan | `firm-brand`, runtime theming everywhere |
| `User` | role (`client` \| `staff`), email, mobile, biometric flag | `ob-*`, `firm-clients` |
| `Receipt` | image/PDF ref, merchant, date, amount, gst, category, `ocr_confidence`, status | `capture`, `rec-receipts`, `home` |
| `VehicleTrip` | date, from/to, purpose, km, business flag; + logbook period & business-use % | `rec-log` |
| `WFHEntry` | date, hours (contemporaneous daily diary) | `rec-wfh` |
| `Category` | ATO-aligned seed list per tenant | `capture`, `rec-receipts` |
| `Thread` / `Message` | participants, body, read receipts, attachments | `inbox` |
| `DocumentRequest` | label, due, status (the "Requested: …" chips) | `inbox` |
| `ReturnStatus` | stage: received → in progress → ready to sign → lodged | `inbox` tracker |
| `ContentPost` | title, body, image, type (article/event), publish state | `hub`, `firm-content` |
| `Event` | title, datetime, RSVP list | `hub` |
| `PushCampaign` | message, segment, schedule, frequency-cap state | `firm-send` |
| `Subscription` | Stripe ids, plan, status | provisioning |
| `EngagementEvent` | user, type, ts → powers churn/login-decline scoring | `firm-insights` |
| `ExportBundle` | date range, CSV+PDF+image-zip ref, delivered-at (audit trail) | records export / hand-off |

**Tax constants** (`packages/shared`): cents-per-km ($0.88, 2025–26, max 5,000 km), WFH fixed
rate (70c/hr, 2025–26), $300 written-evidence threshold — all **versioned by financial year** so
historical records compute correctly.

---

## B4. MVP scope vs. deferred

**In MVP**
- Passwordless onboarding + biometric + per-tenant runtime theming (`ob-*`).
- Capture: receipt photo → OCR → AI suggestion card → confirm (`capture`); email-forward + SMS fallback channels.
- Records: receipts list, vehicle logbook (12-week % **and** cents-per-km), WFH hours diary with live "hrs × 70¢ = $X" (`rec-*`).
- Export/hand-off bundle (CSV + PDF + image zip) with delivery audit trail.
- Secure messaging + document requests + return status tracker (`inbox`).
- Push: transactional + AU-tax-calendar reminders, segmented, with frequency cap (`firm-send`).
- Firm content feed + events + "Ask your accountant" Q&A (`hub`).
- Tax-Readiness ring + live $ claimable + prep streak (`home`).
- Firm console: branding self-serve, CSV import + magic-link/SMS invite, push composer, content/AI-newsletter, churn/at-risk dashboard (`firm-*`).
- Stripe self-serve subscription + auto tenant provisioning.

**Deferred (fast-follow / later)**
- Xero/MYOB sync; TFN handling (**avoid entirely in MVP**); open community forums;
  any "specific-advice" AI; per-firm store binaries (premium tier only, à la TaxDome).

---

## B5. Milestones

Each milestone realizes specific prototype screen IDs so progress is visible against the mockup.

| # | Milestone | Realizes |
|---|---|---|
| **M0** | Repo scaffolding, CI, multi-tenant DB skeleton (RLS), shared theme tokens, docker-compose dev env | — |
| **M1** | Passwordless auth + biometric + onboarding + runtime per-tenant theming | `ob-welcome`, `ob-identify`, `ob-sent`, `ob-faceid`, `ob-home` |
| **M2** | Capture + OCR/AI confirm + records (logbook + WFH) + offline sync + export bundle | `capture`, `rec-receipts`, `rec-log`, `rec-wfh` |
| **M3** | Secure messaging + document requests + return status tracker + transactional push | `inbox` |
| **M4** | Firm console: branding, CSV import/invite, push composer, content library + AI newsletter, churn dashboard | `firm-clients`, `firm-invite`, `firm-brand`, `firm-send`, `firm-content`, `firm-insights` |
| **M5** | Tax-Readiness ring + live $ + streaks/milestones + tax-calendar scheduler + Stripe self-serve | `home`, `hub`, `assistant` |

---

## B6. Compliance build checklist

Derived from the AU constraints in the research; these are also trust/marketing assets.

- [ ] **5-year durable, exportable storage** — object-store lifecycle ≥5 yrs; export bundle always available.
- [ ] **AU data residency** — DB, object storage, and any LLM processing in AU regions.
- [ ] **No TFN** — schema and intake deliberately exclude TFNs; hand off to the registered agent.
- [ ] **AI-use disclosure + human-in-the-loop** — AI suggestions are confirmable, never auto-committed; "general info, not advice — your accountant confirms" copy (prototype `assistant` disclaimer).
- [ ] **APP automated-decision disclosure** (live 10 Dec 2026) — privacy policy names AI categorisation/Q&A with human-review/opt-out.
- [ ] **Tax constants versioned by FY** — $0.88/km, 70c/hr, $300 threshold; WFH requires contemporaneous full-year hours (no estimates).
- [ ] **Push discipline** — frequency cap (≤3/client/month), per-topic + entity/deadline segmentation, granular opt-out.
- [ ] **External-model contracts** — no-training/no-retention with OCR/LLM vendors.

---

## B7. Verification (for the build)

**Local run**
1. `docker compose up` → Postgres + Redis + MinIO.
2. `backend`: run migrations, seed a demo tenant (Acme Advisory) + a client (Sam Carter).
3. `npx expo start` for `app`; `next dev` for `console`.

**End-to-end happy path to exercise**
- Magic-link sign-in → biometric enrol → onboarding "do-this-now" home.
- Capture a receipt → OCR returns fields + confidence → confirm → appears in records + ring/$ update.
- Log a vehicle trip (incl. conversational "I drove 40km…") and WFH hours (live $ recompute).
- Build an export bundle → verify CSV+PDF+zip and the delivery audit record.
- Firm console: change brand colour → app re-themes; CSV import → magic-link invites queued; schedule a tax-calendar push → scheduler dry-run fires; at-risk dashboard flags the quiet client.
- Stripe test subscription → tenant auto-provisioned.

**Tests**
- Unit: Jest (tax-constant math, OCR-field mapping, segmentation/frequency-cap logic, RLS scoping).
- E2E: Detox (onboarding → capture → confirm → export).
- Tenant isolation: automated test that one tenant cannot read another's rows (RLS).

**For the prototype (now):** open `prototype/index.html` (or the live Pages URL) — confirm the
Onboarding/Client/Firm switch, that the brand swatch re-themes the whole UI, and that every screen
ID listed in M1–M5 is reachable.
