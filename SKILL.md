---
name: waguri-captcha-solver
description: "Use when solving Turnstile, reCAPTCHA, hCaptcha, Cloudflare, AWS WAF, BotGuard, DataDome, PerimeterX, Akamai, Aliyun, or Arkose captchas. Local HTTP sidecar on CloakBrowser."
version: 1.0.0
category: automation
---

# waguriagentic/captcha-solver

Local captcha-solving HTTP sidecar built on `cloakbrowser` (self-hosted, anti-detect Chromium). Solves challenges natively in a real browser engine — no external per-solve cost for browser-based paths. Harvest-only: returns replayable tokens; account creation/form recon is the caller's job.

## Supported Types (11)
Turnstile, reCAPTCHA (v2/v3/invisible/Enterprise), hCaptcha (checkbox/invisible/real-page), Cloudflare (`cf_clearance`), AWS WAF (`aws-waf-token`), BotGuard (Google OAuth `bgRequest`), DataDome (`datadome` cookie), PerimeterX (`_px3` via press-hold PoW), Akamai (`_abck` via bmak sensor), Aliyun (slide-puzzle token), Arkose/FunCaptcha (ONNX visual puzzle).

## Setup
```bash
pip install -r requirements.txt
./run.sh
```
Server: FastAPI on `:8877`. Auth: Bearer token from `.solver-token.env` (chmod 600) for `/solve` and `/logs`.

## Endpoints
| Method | Path | Auth | Description |
|---|---|---|---|
| GET | `/health` | public | Liveness + supported types |
| GET | `/logs` | token | Last N solve events |
| POST | `/solve` | token | Solve a captcha (dispatch by `type`) |

```bash
curl -X POST http://127.0.0.1:8877/solve \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type": "turnstile", "url": "https://target.cc/", "sitekey": "0x4AAAAA..."}'
```

## Layout
- `server.py` — FastAPI dispatcher, SSRF guard, global timeout
- `common/` — shared CloakBrowser utils
- `<provider>/solve.py` — per-provider solvers
- `arkose/models/` — 24 ONNX models (~1.4GB) for FunCaptcha visual puzzle

## Pitfalls
- Arkose needs the ONNX models dir (~1.4GB) — don't strip it.
- reCAPTCHA runs headed (needs Xvfb on headless VPS).
- DataDome is site-agnostic: caller passes URL + referer (e.g. GitHub octocaptcha gate).
- PerimeterX needs real CDP press-hold gesture; it drives the SHA-256 hashcash PoW worker + biomechanics.