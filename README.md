# 🦉 OJack

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Linux%20%2F%20Windows-informational?style=flat-square&logo=linux&logoColor=white&color=0a0c10"/>
  <img src="https://img.shields.io/badge/Category-ORecon%20%2F%20OWeb-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat-square"/>
  <img src="https://img.shields.io/badge/Part%20of-OwlSec%20Toolkit-7b5ea7?style=flat-square"/>
  <img src="https://img.shields.io/badge/Version-2.0-cyan?style=flat-square"/>
</p>

> **OJack** is a Clickjacking vulnerability scanner — it analyses `X-Frame-Options` and `Content-Security-Policy` response headers, computes a risk score, generates a live iframe PoC test page, and exports structured JSON reports.

---

> ⚠️ **AUTHORISED SECURITY TESTING ONLY** — Use only on systems you own or have explicit written permission to test.

---

## 📌 Overview

OJack fetches HTTP response headers from a target URL, evaluates Clickjacking protection across two security headers, predicts browser framing behaviour, and correlates the header analysis with an optional real observed result from the PoC iframe test — producing a final verdict with actionable intelligence hints.

---

## 🖥️ Menu

| Option | Description |
|--------|-------------|
| **[1] Single URL Scan** | Fetch headers → analyse XFO + CSP → risk score → optional observed behavior correlation |
| **[2] Scan + PoC Page** | Full scan + generate iframe HTML test page → open in browser → record observed result |
| **[3] Batch Scan** | Scan multiple URLs from a `.txt` file — risk score, XFO status, CSP status per URL |
| **[4] Last Results** | View all scan results from the current session |
| **[5] Export Results** | Save full session results to a JSON report |

---

## 🔍 Detection Logic

### X-Frame-Options (XFO)

| Header Value | Status | Meaning |
|-------------|--------|---------|
| `DENY` | 🟢 Strong | Blocks all framing |
| `SAMEORIGIN` | 🟢 Strong | Same-origin framing only |
| `ALLOW-FROM <origin>` | 🟠 Weak | Deprecated — ignored by modern browsers |
| Not set | 🔴 Missing | No XFO protection |

### Content-Security-Policy (CSP) — `frame-ancestors`

| Directive Value | Status | Meaning |
|-----------------|--------|---------|
| `'none'` | 🟢 Strong | Blocks all framing |
| `'self'` | 🟢 Strong | Same-origin framing only |
| Specific origins | 🟢 Strong | Allowlist enforced |
| `*` (wildcard) | 🟠 Weak | Allows any origin to frame |
| `frame-ancestors` missing from CSP | 🟠 Weak | CSP present but no framing control |
| Not set | 🔴 Missing | No CSP protection |

---

## 📊 Risk Score

Scored 0–10 based on header evaluation and observed browser behavior:

| Score | Level | Condition |
|-------|-------|-----------|
| 0–3 | 🟢 **LOW** | Strong protection in place |
| 4–5 | 🟡 **MEDIUM** | Partial or weak protection |
| 6–7 | 🟠 **HIGH** | Missing protection headers |
| 8–10 | 🔴 **CRITICAL** | No protection + iframe loaded confirmed |

Score adjustments: +3 if observed as Loaded · -2 if observed as Blocked · +1 for Blank/Redirected · +2 for missing XFO · +2 for missing CSP · -2 if any header is Strong.

---

## 🌐 PoC Page

**[2] Scan + PoC Page** generates a labeled, non-deceptive iframe test HTML file saved to `oclick_output/`:

```
oclick_output/oclick_poc_<domain>.html
```

The page opens in the default browser and displays the target inside an `<iframe>`. The analyst observes the result and selects from:

| Observed Result | Meaning |
|----------------|---------|
| **Loaded** | Target rendered inside iframe — framing is possible |
| **Blocked** | Browser refused to load — protection working |
| **Blank** | Empty frame — login wall, JS frame-busting, or mixed content |
| **Redirected** | Target redirected — needs further manual testing |

---

## 🧠 Verdict & Hints

After combining header analysis with observed behavior, OJack produces a verdict:

| Scenario | Verdict |
|----------|---------|
| Loaded + strong headers | `HIGH ALERT — header mismatch (possible bypass / header stripping)` |
| Loaded + weak/missing headers | `VULNERABLE — framing confirmed` |
| Blocked + missing headers | `INCONSISTENT — browser or extension may be blocking` |
| Blocked + strong headers | `PROTECTED — blocking consistent with headers` |
| Blank or Redirected | `INCONCLUSIVE — needs manual confirmation` |
| Headers only (no browser test) | `PROBABLY PROTECTED / POSSIBLE RISK / HIGH RISK` |

Each result includes **Intelligence Hints** — contextual analysis tips such as checking CDN header stripping, testing from hosted origins, and verifying across multiple browsers.

---

## 📤 Reports

All outputs are saved to `oclick_output/`:

| File | Contents |
|------|----------|
| `oclick_poc_<domain>.html` | Non-deceptive iframe PoC test page |
| `oclick_report_YYYYMMDD_HHMMSS.json` | Full session report with all scan results |

Each JSON entry contains URL, final URL (after redirects), HTTP status, XFO status, CSP status, prediction, observed behavior, risk score, risk label, verdict, and timestamp.

---

## ⚙️ Requirements

- **Linux or Windows**
- **No Python installation needed** — runs as a standalone executable

---

## 🚀 Usage

```bash
./OJack
```

---

## 📦 Part of OwlSec Toolkit

This tool is part of the **OwlSec** suite — a collection of 300+ security and privacy tools.

🔗 [owlsec.org](https://owlsec.org)

---

## ©️ License

MIT License — © Khaled S. Haddad

*Tools are distributed as pre-built executables. Source code is proprietary.*
