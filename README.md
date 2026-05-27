# Provably-Fair Roll Verifier

Static page that lets players re-compute any roll from the bot's PF stream
**entirely in their own browser** — no server, no outside scripts. Just open
`index.html` (or visit the hosted URL) and paste in the seed/client/nonce/range
fields, or follow a pre-filled link from the bot's `,verifylink` / `.verify`
commands.

## What it does

Mirrors the bot's `verify_roll` function exactly:

```
h    = HMAC_SHA512(server_seed, "{client_seed}-{nonce}")
frac = int(h[:13], 16) / 2**52
roll = int(frac * (max - min + 1) + min)   # raw frac when range is 0-1
```

Also checks the server-seed commitment: `SHA256(server_seed)` must match the
hash the bot showed before the round started (`,serverseed`).

## Deploying to GitHub Pages

1. Create a **public** repo (any name — e.g. `pf-verify`).
2. Upload **both** `index.html` and `info.html` to the repo root.
   (`info.html` is the "per-game nonce breakdown" sub-page that `index.html`
   links to.)
3. Settings → Pages → Source = `Deploy from a branch`, branch = `main`, folder = `/ (root)`.
4. After ~30s the page is live at `https://<user>.github.io/<repo>/`.
5. Set that URL as `VERIFIER_URL` in `cogs/new_verify.py:51`.

That's it. The pages have no build step, no dependencies, no external assets —
everything is inline. Web Crypto API (built into every modern browser) handles
HMAC-SHA512 + SHA256.

## Trust model

Right-click the live page → **View Source** to confirm:
- No `<script src="...">` referencing external URLs.
- No `fetch` / `XMLHttpRequest` calls (the page never phones home).
- All math is inline + uses `crypto.subtle` (the browser's audited primitives).

If the page changes, the URL still works but you should re-verify the source —
a malicious deploy could leak seeds. Keep the repo public so anyone can audit.
