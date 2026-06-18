# reimagined-garbanzo — accountant client app (prototype + MVP plan)

A **white-label, firm-branded client app + firm console** for small Australian accounting
practices (≤ ~500 clients each) — a year-round **retention & engagement** channel, not just a
receipt scanner. Inspired by H&R Block's ReceiptHub, but the *firm* (not a national brand) is the
centre of gravity.

> This repo currently holds the **clickable prototype** and the **MVP build plan**. (The intended
> home is a dedicated `accountant-app` repo; it lives here for now because that repo isn't yet
> reachable from the build environment.)

## What's here

| Path | What it is |
|---|---|
| [`prototype/index.html`](prototype/index.html) | Clickable calm-fintech prototype — client app + firm console, with live white-label theming. |
| [`prototype/README.md`](prototype/README.md) | How to open the prototype and what to try. |
| [`docs/MVP_BUILD_PLAN.md`](docs/MVP_BUILD_PLAN.md) | The concrete engineering plan to build the MVP — stack, data model, scope, milestones, compliance, verification. |
| `.github/workflows/pages.yml` | Deploys the prototype to GitHub Pages. |

## Live prototype

Once GitHub Pages deploys (the workflow auto-enables it):
**https://pareenvatani23.github.io/reimagined-garbanzo/**

## Product in one line

Retention-as-a-service for AU accounting firms: ReceiptHub-parity capture (receipts + vehicle
logbook + work-from-home hours) **standalone**, plus passwordless onboarding, compliant messaging,
tax-calendar push, and a firm content feed — priced flat per practice (anti-Dext-per-client),
shipped as one multi-tenant React Native/Expo app themed per firm at runtime.

See [`docs/MVP_BUILD_PLAN.md`](docs/MVP_BUILD_PLAN.md) for the full plan.
