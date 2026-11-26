# VaultDE — Project Handover Document

> **Privacy Notice:** This repository and handover contain **zero personal financial data**. All user balances, tax details, and advisor PDFs live strictly on the user's device (encrypted `localStorage` + local `IndexedDB`).

---

## 1. Project Overview & Live URLs
- **App Name:** **VaultDE** — Ultra-Private German Personal Finance, Retirement & Advisor Cockpit (PWA)
- **GitHub Repository:** `https://github.com/fesassi-cyber/vaultde-finance` (branch: `main`)
- **Live Pixel PWA (GitHub Pages):** `https://fesassi-cyber.github.io/vaultde-finance/`
- **Primary Device:** Google Pixel (installed as a standalone Chrome PWA) + desktop browser.
- **Latest Code Commit (before handover):** `121e8c5` (`sw.js` cache version `vaultde-pwa-v12`).

---

## 2. Architecture & Repository Layout

```text
fesassi-cyber/vaultde-finance/
├── index.html                    # Single-file SPA (~3,380 lines): UI + Crypto + Tax/Projection Engine + Universal Importer
├── src/input.css                 # Tailwind v4 source + light/dark semantic CSS variables (--background, --card, --chart-1..5)
├── manifest.json                 # Android/Pixel PWA manifest (standalone display, maskable icons)
├── sw.js                         # Offline-first Service Worker (bump CACHE_NAME on every deploy!)
├── icon-192.png                  # Generated PWA icon (192x192)
├── icon-512.png                  # Generated PWA icon (512x512)
├── .github/workflows/pages.yml   # 2-job GitHub Actions workflow (build Tailwind v4 CSS -> deploy GitHub Pages)
├── README.md                     # Repo overview & Pixel install instructions
└── HANDOVER.md                   # This handover document
```

### Build & Deployment Pipeline
- **Zero External CDNs in Production:** GitHub Actions (`.github/workflows/pages.yml`) runs `tailwindcss@4` + `@tailwindcss/cli@4` to compile `src/input.css` (using `@source "../index.html"`) into `./dist/styles.css` (`--minify`) and deploys `./dist` to GitHub Pages.
- **Important Deployment Rule:** Whenever `index.html` or `src/input.css` is updated, **always bump `CACHE_NAME` in `sw.js`** (currently `vaultde-pwa-v12` → next `vaultde-pwa-v13`) so the installed Pixel PWA pulls the latest version cleanly.
- **Pre-push Verification:** Always validate the inline `<script>` in `index.html` with `node --check` before pushing.

---

## 3. Security & Privacy Invariants (Must Preserve)
1. **Zero Server Leakage:** Never introduce a backend, telemetry, analytics, or external CDNs.
2. **Strict Browser CSP (in `index.html` `<head>`):**
   - Allows only `'self'` plus explicit anonymous public market endpoints: `https://api.frankfurter.app` (EUR/USD) and `https://api.gold-api.com` (`XAU`, `XAG`, `XPT`, `XPD`).
   - Zero personal data is ever included in any outbound request.
3. **On-Device Encryption:**
   - Web Crypto API `PBKDF2-SHA256` (250,000 iterations) + `AES-256-GCM`.
   - Export/Import uses encrypted `.vault.json` files.
   - Lock screen includes a 2-tap **Start Over** recovery button (`startOverVault()`) in case decryption fails or a passphrase is forgotten.
4. **Shoulder-Surfing Protection:** `👁 Hide €` blurs all `.sensitive` elements.
5. **Local PDF Storage:** Advisor PDFs (Daniel Bossenz) are stored as binary blobs in browser `IndexedDB` (`VaultDE_PDFs_v1`), never uploaded anywhere.

---

## 4. Functional Map (5 Tabs)

| Tab | Purpose & Key Features |
| :--- | :--- |
| **1. Status & Cashflow** | Top 4 KPIs (Net Worth + QoQ delta, Monthly Net Salary, Monthly Surplus/Sparquote, Liquidity Runway); **💡 Your Finances in Plain English** 3-card coach; **5-Pillar Financial Health Score (0–100)**; **How Fast Your Wealth Grows** (Contributions vs. Passive Growth + **In 2 / 5 / 10 Years** forecast + next milestone ETA); **Usable Now vs. Reserved for Old Age** + paycheck split; Independent **Gross Annual Brutto & Monthly Netto** (unlinked by default, optional `Auto-link ⇄` checkbox); Retirement Year badge; Monthly Save & Invest Plan (`Tagesgeld`, `Festgeld`, `FFB+Solit`, `bAV+Conti`); Allocation Donut; Quarterly Net Worth Sparkline. |
| **2. Retirement** | **Today vs. Your Plan Hero Scorecard** (Traffic-light verdict, **If You Stopped Saving Today** vs. **If You Follow Your Plan to Age X · Year Y**, visual Target Thermometer, replacement ratio); 4 cards: **DRV**, **MetallRente (bAV)**, **Continentale Zusatzrente** (Schicht 1/2/3 tax selector + optional policy payout override), and **Rentenlücke** with **`Include FFB ETFs in retirement payout` OFF by default**; 4-Pillar Income Mix bars; **Post-Retirement Drawdown Simulator (Age 67 → 95)**. |
| **3. Investments** | **Anonymous Market Oracle** (live `EUR/USD` + 4 metals `XAU/XAG/XPT/XPD` in €/g); **Freistellungsauftrag & ETF Vorabpauschale** tracker; editable **FFB Konto & ETF Sparplan** table; **Solit Edelmetalle** (Gold, Silver, Platinum/Platin, Palladium in grams + monthly Sparplan); **Festgeld & Tagesgeld** (Stand, interest %, monthly save plan). |
| **4. Plans, Predictions & Strategy** | Interactive **What-If Sandbox** sliders (Extra ETF Sparplan, Retire Age, SWR); hover/touch-scrub **Wealth Forecast Chart** (Conservative / Base / Optimistic + Contributed Principal dotted line, Real vs. Nominal toggle); **Future Milestones Breakdown Table** (`+2, +5, +10, +15, +20 yrs, Retire`); editable **Financial Goals** with dynamic ETA; On-Device **Smart Strategy Engine**. |
| **5. Advisor (D. Bossenz)** | Drag-and-drop local **IndexedDB PDF Vault** with inline PDF viewer & download; searchable/categorized briefings archive; **Advisor Action Checklist** (surfaces a reminder banner on Tab 1 when items are open). |

---

## 5. User Preferences & Domain Decisions (Do Not Regress!)
- **Layperson-Friendly Language:** User explicitly prefers clear, everyday explanations without heavy finance jargon. Keep labels plain and human-readable.
- **Brutto & Netto Decoupled by Default:** User's actual payslip numbers are more precise than any tax estimator. Keep `linkBruttoNetto: false` by default so neither field overwrites the other unless the user explicitly checks **Auto-link ⇄**.
- **FFB ETFs Excluded from Retirement Payout by Default:** User plans to spend FFB ETFs along the way. Keep `includeFfbInRetirement: false` by default (still counted in Total Net Worth and 2/5/10-year wealth forecasts).
- **No "Reset to Demo" Button:** Removed by request to prevent accidental data loss.
- **Quarterly Maintenance Workflow (1x/Q):**
  - Fast 60-second **📅 1x/Q Update** modal for balances that change quarterly.
  - **✨ Gemini Import** uses a permanent private **Gemini Gem** (`VaultDE Quarterly Extractor`). VaultDE's `parseUniversalGeminiInput()` accepts **both** plain-text `KEY = VALUE` lines (recommended) **and** auto-repaired messy JSON, with a **🔍 Test & Preview** button.
- **All 4 Solit Metals Supported:** Gold, Silver, Platinum (Platin), and Palladium.

---

## 6. How to Continue in the New Project
1. Attach `fesassi-cyber/vaultde-finance` to the new project.
2. In your first message in the new project, simply tell the agent:
   > *"Read `HANDOVER.md` in `fesassi-cyber/vaultde-finance` and continue from there."*
3. Remember: after any code edit, validate inline JS (`node --check`), bump `CACHE_NAME` in `sw.js`, push to `main`, and verify the `Build & Deploy VaultDE PWA to GitHub Pages` GitHub Actions run succeeds.
