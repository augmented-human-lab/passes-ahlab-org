# passes.ahlab.org — Visitor Passes console

Admin console for AHL **Visitor Passes**, the printed credit-card cards that give a visitor an
`@ahlab.org` guest account. Static site on GitHub Pages; the backend is the
[ahl-visitor-form](https://github.com/augmented-human-lab/ahl-visitor-form) Apps Script.

## What admins do here

- **Activate a pass at handover.** Type the code under the QR (desktop) or tap **Scan** to
  read the QR with the camera (phones). Optionally note who it was given to. The holder then
  has 7 days to claim their account; an unclaimed pass switches back off by itself.
- **Track every pass:** inactive, active, used, revoked and expired, with search, a
  per-batch filter and a history for each pass (created → activated by/for → claimed → revoked).
- **New sheet:** creates 8 inactive passes and opens the print view (Print, Download PDF,
  or the PLT cut-line file with cutter registration marks). **Print batch** reprints any batch.
- **Deactivate, revoke, extend** a pass from its details.

Printed passes start **inactive**. Scanning one before it's activated shows "Pass Not
Activated", so a lost card is useless until an admin activates it.

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
