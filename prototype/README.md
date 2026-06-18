# LedgerLink — interactive prototype

A single self-contained HTML file (`index.html`, no build step, no dependencies) that
demonstrates the white-label **client app** and **firm console** in a calm-fintech visual
language. Once Pages is enabled it is live at
`https://pareenvatani23.github.io/reimagined-garbanzo/`.

## How to open

- **Live:** the GitHub Pages URL above (published automatically by the GitHub Actions workflow when `main` updates).
- **Locally:** open `prototype/index.html` in any browser — no server required.

## Demo it on Android

This prototype is an **installable PWA**, so you can showcase it to clients as a full-screen app —
no app store, no `.apk`, no build:

1. On the client's (or your) Android phone, open **https://pareenvatani23.github.io/reimagined-garbanzo/** in **Chrome**.
2. Tap the **⋮ menu → _Install app_** (or _Add to Home screen_). Accept.
3. Launch it from the new home-screen icon — it opens **full-screen with no address bar**, looking and
   feeling like a native app, and still works offline (a service worker caches it).

Other ways to show it:

- **iPhone/iPad:** Safari → **Share → Add to Home Screen** (same full-screen result).
- **Laptop / screen-share:** open the URL in Chrome and toggle **DevTools → device toolbar** (Ctrl/Cmd+Shift+M)
  to show it in a phone frame; or run it inside **Android Studio's emulator** (Device Manager → create a
  virtual device → open Chrome → load the URL) if you want a true Android shell on your desktop.
- **Real installable APK (later):** because it's a valid PWA, you can wrap the live URL into a signed
  Android package with **[PWABuilder](https://www.pwabuilder.com/)** or **Bubblewrap** when you want a
  sideloadable/Play-Store build.

> Note: this is still the **web prototype**. The production native Android app (React Native + Expo, run
> via Expo Go / EAS builds) is the long-term path — see [`../docs/MVP_BUILD_PLAN.md`](../docs/MVP_BUILD_PLAN.md).

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
