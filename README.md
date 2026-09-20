# RC License Provisioner (HTML)

A single-file, browser-based tool for RingCentral partner operations — no build step, no server, no dependencies. Open `rc-license-provisioner.html` directly in a browser.

It's a browser port of the native "RC License Provisioner" macOS app, extended with account-creation flows for both RingEX (REX) and RingCX (RCX).

## Features

The app has three tabs, sharing a common Partner and Environment selector:

### 1. Create Account (RingEX)
Creates a brand-new RingEX account via the partner-level `initial-order` API.
- Partner: Cox, Charter SMB, or Charter Enterprise
- Package: pulled from the partner's package list (default: **Ultra**)
- Company Name / Email: auto-generated (`ACME_<YYYYMMDD>_<random3>`, lowercase email), with a Regenerate button
- Contact info, address, Main Number (required — RingCentral Phone Number Pool), optional Fax Number
- Default licenses: `LC_ALN_38` x5, `LC_DL-UNL_50` x3
- Status defaults to **Confirmed**
- On success, the returned account ID is used to prefill both the Provision Licenses and Create RCX Account tabs

### 2. Provision Licenses
Adds or removes licenses on an existing RingCentral account.
- Account ID field with quick presets
- Add/Remove action toggle
- Multiple license lines (SKU + quantity + optional cost center), with SKU presets
- Optional auto-open of the account summary page after provisioning

### 3. Create RCX Account (RingCX)
Attaches a RingCX (Contact Center) capability to an existing RingCentral account via `POST /cx/provisioning/v1/accounts/{accountId}/initial-order`.
- Only available for **Cox** and **Charter Enterprise** (Charter SMB is not offered RCX; the UI shows a notice and disables the button)
- Account ID defaults to the ID from the last successful Create Account run
- Package presets per partner (default: **Standard Package - Named**)
- License lines (SKU + quantity), defaulting to `SA_SEAT_4` x1 and `SA_CRSYEAR_26` x1
- Reporting retention period (1–8 years)
- Sub Account Name auto-generated as `RCX_<company name>` (from the Create Account tab), with a Regenerate button

## Authentication

Two distinct OAuth JWT-bearer flows are implemented entirely client-side using WebCrypto (RS256 signing, no server round-trip):

| Flow | Grant type | Used by | Assertion claims |
|---|---|---|---|
| Account-level | `urn:ietf:params:oauth:grant-type:jwt-bearer` | Provision Licenses, Create RCX Account | `sub` = target account ID |
| Brand-level (Partner Access Token) | `partner_jwt` | Create Account (RingEX) | `plid` = uBrand ID, `rtag` = region tag |

Private keys are supplied as PEM (PKCS#1 `RSA PRIVATE KEY` or PKCS#8 `PRIVATE KEY`) via the **Credentials…** dialog, per partner profile:
- Client ID, Key ID (kid), optional JWKS URL (jku), Region tag (rtag)
- RSA private key — paste, load from file, or save to this browser's `localStorage`

**uBrand ID (plid)** is not a credential field — it's derived automatically from the selected Partner (Cox = `3000.Cox`, Charter SMB = `5110`, Charter Enterprise = `5210`).

⚠️ **Private keys saved via "Save Key" are stored in browser `localStorage`, not an OS keychain.** Only use this on a trusted machine; prefer pasting the key fresh each session if unsure.

## Environments

Toggle between **Production** (`platform.ringcentral.com`) and **UAT** (`platform.uat.ringcentral.com`) per partner profile.

## Known limitations

- **CORS**: Direct browser calls to the RingCentral platform API may be blocked by CORS depending on the app/network configuration. If you see a network error, this is the likely cause.
- Package/pricing IDs (REX and RCX) are hardcoded from partner package sheets provided at build time. If pricing catalogs change, update the `PACKAGE_PRESETS` / `RCX_PACKAGE_PRESETS` constants in the script.

## File structure

Everything lives in a single file:
- `rc-license-provisioner.html` — HTML, CSS, and JS (WebCrypto JWT signing, RingCentral API client, UI) in one self-contained document.
