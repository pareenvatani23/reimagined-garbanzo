# LedgerLink — interactive prototype

A single self-contained HTML file (`index.html`, no build step, no dependencies) that
demonstrates the white-label **client app** and **firm console** in a calm-fintech visual
language. Once Pages deploys it is live at
`https://pareenvatani23.github.io/reimagined-garbanzo/`.

## How to open

- **Live:** the GitHub Pages URL above (deployed by `.github/workflows/pages.yml`).
- **Locally:** open `prototype/index.html` in any browser — no server required.

## What to try (demo chrome at the top)

- **View switch — Onboarding / Client app / Firm console.** Toggle between the three surfaces.
- **Firm brand swatches.** Click a colour to re-theme the *entire* UI live. This proves the
  white-label model: the whole app is driven by CSS custom properties (`--brand`, `--brand-ink`,
  `--brand-soft`), so a firm's brand colour is a single runtime variable — no per-firm build.

## Screens (section IDs in the file)

| Surface | Screens |
|---|---|
| Onboarding | `ob-welcome`, `ob-identify` (passwordless magic-link), `ob-sent`, `ob-faceid` (biometric), `ob-home` ("what to do now") |
| Client app | `home` (tax-readiness ring + live $ + streak), `capture` (camera + AI suggestion card + email/SMS fallback), `records` (`rec-receipts` / `rec-log` / `rec-wfh`), `inbox` (status tracker + compliant messaging), `hub` (content feed + events + Q&A), `assistant` (general-info AI with disclaimer) |
| Firm console | `firm-clients`, `firm-invite` (CSV import + magic-link/SMS), `firm-brand` (logo + colour, live preview), `firm-send` (segmented push + frequency cap + tax-calendar schedule), `firm-content` (AI auto-newsletter + library), `firm-insights` (churn / at-risk dashboard) |

The prototype is a clickable design mockup to react to and iterate on — **not** production React
Native code. The engineering plan that turns it into a shippable MVP is in
[`../docs/MVP_BUILD_PLAN.md`](../docs/MVP_BUILD_PLAN.md).
