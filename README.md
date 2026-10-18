# passes.ahlab.org — Visitor Passes console

Admin console for AHL **Visitor Passes**, the printed credit-card cards that give a visitor an
`@ahlab.org` guest account. Static site on GitHub Pages; the backend is the
[ahl-visitor-form](https://github.com/augmented-human-lab/ahl-visitor-form) Apps Script.

## What admins do here

- **Activate a pass at handover:** press **Scan a pass 📷**. The screen opens the camera
  (phones; any device with a camera) and a code box. Scanning a card's QR fills the box, or type
  the code printed under it. The pass's state shows straight away (ready, already active,
  revoked, used, expired, not found) and **Activate** is enabled only when it makes sense
  (**Re-activate** for a revoked pass). Then **Scan next pass**. The holder has 7 days to claim;
  an unclaimed pass switches back off by itself.
- **Row actions:** **Activate** on inactive and revoked passes; **Deactivate** (e.g. undo a test
  activation) and **Revoke** (second click confirms) on active passes.
- **Track every pass:** inactive, active, used, revoked and expired, with search, a
  per-batch filter and a history for each pass (created → activated by → claimed → revoked), and a copy button next to each code.
- **New passes set:** creates 8 inactive passes and opens the print view (Print, Download PDF,
  **PLT (Preview)**: a PDF of just the cut lines + registration marks to check, or **PLT**: the
  real `.plt` cutter file, one per A4 sheet).
- **Passes sets:** every set created so far, newest first, with its card counts and an
  **Open print sheet** button (prints its still-usable passes) plus **Show passes**.
- **Deactivate, extend, revoke or re-activate** a pass from its details (click the row).

Printed passes start **inactive**. Scanning one before it's activated shows "Pass Not
Activated", so a lost card is useless until an admin activates it.

## The visitor claim page (`/claim/`)

QR codes on new cards point at `https://passes.ahlab.org/claim/?t=AHLV-XXXX-XXXX`. The page
asks the backend about the pass (`claimInfo`), then shows the sign-up form (active pass)
or why it can't be claimed (not activated, used, revoked, expired, unknown). Submitting calls
`claim`, which runs the same server code as before: validation, lock, account, emails.
Neither action needs sign-in; the pass code is the credential.

Why not link straight to the Apps Script web app: when a phone is signed into several Google
accounts, Google rewrites `script.google.com/macros/s/…` to `…/macros/u/1/s/…`, and that URL
fails with "Sorry, unable to open the file at present" before the script runs. Cards printed
before this change still point at the script URL. They work for most people, but reprint
any that haven't been handed out.

`claim/index.html` is generated from ahl-visitor-form `src/Index.html` (same look, wording and
validation). Keep the two in sync.

## How it fits together

```text
passes.ahlab.org ──sign in──► AHL member-login broker (ahl-site-appscript)
       │                         └─► /auth-callback/  (session token, 7 days, localStorage)
       │
       └──POST JSON (text/plain, no preflight) {action, auth: <session token>}──►
             ahl-visitor-form /exec  doPost → verifies the token's HMAC with
             BROKER_HMAC_SECRET + checks CONFIG.adminEmails → Tokens sheet
```

- Access control lives in the backend: every API call needs a valid broker session token
  for an address in `CONFIG.adminEmails` (ahl-visitor-form `src/Config.js`). This repo
  holds no secrets.
- `localhost` / `127.0.0.1` talk to the **dev deployment** of the backend; everything else
  talks to the production `/exec`. Both URLs are at the top of the script in `index.html`
  and `print.html`.
- `print.html` is generated from ahl-visitor-form's `src/Cards.html`, with its data loaded
  from the API instead of injected by Apps Script. Keep the two in sync when changing card
  artwork.

## Local development

```bash
python3 -m http.server 8795 --bind 127.0.0.1    # → http://localhost:8795/
```

Port 8795 is allow-listed as a sign-in return URL in the broker. Sign-in works locally,
against the dev backend deployment.

## Share metadata

`og-image.png` is rendered from `og-card.html` (command in that file). Icons: `icon.svg` →
`icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png`.

## The PLT cutter file

`print.html` → `sheetPlt()` writes the same HPGL dialect as the sclab flatbed cutter's Kerf
app (`flatbed-cutter-control/app/lib/hpgl.js`), decoded from SignMaster's working files:
`TB25,11080,7600;CT1;` + 1025 spaces + `IN;`, one `PU`/`PD` per point, `PU11280,200;PG;` + CRLF.
40 units/mm, bed 277 × 190 mm, origin bottom-left, Y up. Portrait A4 is rotated 90° and
centred, which puts the four registration dots on the bed corners:
`X = (287 − y)·40`, `Y = (200 − x)·40` (A4 mm, Y down). Output is identical, point for point,
to running the same outlines through Kerf's own generator.

Unverified on the machine: the direction of that 90° turn (the SignMaster samples were all
symmetric). A full 8-card sheet is symmetric, so it cuts correctly either way; check a
part-filled sheet before relying on it.
