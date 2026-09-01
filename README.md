# omarchy.az

Domain-for-sale landing page. Single static HTML file, no build step, no dependencies.

## Before it works

The contact form is inert until you add a Web3Forms access key:

1. Go to <https://web3forms.com> and enter your email — they mail you a key, no account needed.
2. In `index.html`, replace `WEB3FORMS_ACCESS_KEY` with that key.
3. Commit and push.

Until then the form shows an explicit "not connected yet" message rather than
failing silently. The key is public in page source by design — it only permits
sending mail to the address you registered with.

## Hosting — GitHub Pages

Repo must be **public** (Pages from a private repo requires a paid plan).

**Settings → Pages:** source `main`, folder `/ (root)`, custom domain `omarchy.az`.
Once the certificate is issued, tick **Enforce HTTPS**. Issuance can take up to
24 hours; the site is reachable over plain HTTP before then.

The `CNAME` file in this repo is what GitHub matches incoming requests against —
that is how this site is told apart from any other Pages site on the account.

## DNS — set at online.az

Four **A** records on the apex (`@`):

    185.199.108.153
    185.199.109.153
    185.199.110.153
    185.199.111.153

One **CNAME** for `www`:

    www  ->  AliKhudiyev.github.io.

Pointing `www` at the same `AliKhudiyev.github.io` target as any other Pages site
is correct and expected — routing is decided by the `CNAME` file, not the DNS target.

## Verify

    dig omarchy.az +short          # expect the four IPs above
    dig www.omarchy.az +short      # expect the CNAME
    curl -sI https://omarchy.az    # expect 200, valid cert

## Local preview

    python3 -m http.server 8000

Then open <http://localhost:8000>.

## Deploy

Push to `main`. That's the whole deploy.
