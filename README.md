# VaultDE — Ultra-Private German Personal Finance PWA

**VaultDE** is a 100% client-side, zero-server-leakage Progressive Web App (PWA) for German personal finance, retirement planning (DRV & MetallRente), investments (FFB ETFs, Solit Edelmetalle, Tagesgeld/Festgeld), and advisor briefings.

## Privacy & Security Architecture
- **Zero Personal Data on GitHub**: This repository contains only the empty application code and static assets.
- **On-Device Encryption**: Local vault data can be locked with **Web Crypto API `PBKDF2-SHA256 (250,000 iterations) + AES-256-GCM`**.
- **Strict Content Security Policy (CSP)**: Browser-enforced CSP blocks all unauthorized outbound traffic. Only explicit anonymous public market syncs (`api.frankfurter.app`, `api.gold-api.com`) are permitted when you tap **Sync Public Rates**.
- **100% Offline Capable**: All HTML, compiled CSS, and icons are cached locally by a Service Worker (`sw.js`) — works in Airplane Mode.

## Install on Pixel / Android
1. Open **https://fesassi-cyber.github.io/vaultde-finance/** in Chrome on your Pixel.
2. Tap **📲 Install App** in the top bar (or Chrome menu **⋮ → Add to Home screen / Install app**).
3. Launch **VaultDE** directly from your home screen with full offline & AES-256 encryption support.
