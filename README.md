# omarchy.az

Domain-for-sale landing page. Single static HTML file, no build step, no dependencies.

Live at <https://omarchy.az>.

## Who does what

Four services are involved. Knowing which is which saves a lot of time when
something breaks:

| Service | Role |
|---|---|
| **online.az** | Registrar. Holds the registration and the **nameserver delegation only**. |
| **deSEC** (desec.io) | DNS zone. The A and CNAME records actually live here. |
| **GitHub Pages** | Hosting, served from `main` in this repo. |
| **Web3Forms** | Contact-form backend. Emails each submission. |

The easy mistake: **there are no DNS records at online.az.** It only points the
domain at deSEC's nameservers. Go to deSEC to change anything about resolution.

## DNS

**At online.az** — nameserver delegation only:

    ns1.desec.io
    ns2.desec.org

**At deSEC** — the actual zone:

| Subname | Type | TTL | Records |
|---|---|---|---|
| *(empty)* | `A` | 3600 | `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153` |
| `www` | `CNAME` | 3600 | `AliKhudiyev.github.io.` |

Empty subname is the apex. The four A records are GitHub's redundant Pages
servers — all four, not alternatives. Pointing `www` at `AliKhudiyev.github.io.`
is correct even though other Pages sites use the same target; GitHub routes on
the request's host, matched against the `CNAME` file in each repo.

## Hosting

Repo must be **public** (Pages from a private repo requires a paid plan).

**Settings → Pages:** source `main`, folder `/ (root)`, custom domain
`omarchy.az`, **Enforce HTTPS** on.

## Certificate

Nothing to do. GitHub issues and **auto-renews** a Let's Encrypt certificate.
Those are 90-day certs, renewed automatically well before expiry — the date you
see in a browser is not a deadline.

Renewal validates over HTTP through the domain, so it only fails if resolution
breaks first: delegation removed at online.az, the zone deleted at deSEC, or the
A records changed. Fix DNS and the certificate recovers on its own.

The real expiry risk is **the domain registration at online.az**, not the
certificate. If that lapses, everything stops.

## Contact form

Wired to Web3Forms. The access key is the hidden `access_key` input in
`index.html`. It is public by design — it only permits sending mail to the
address it was registered with, so it is safe in a public repo. Regenerate it at
web3forms.com if it ever starts attracting spam.

## Verify

    dig omarchy.az NS +short              # expect ns1.desec.io, ns2.desec.org
    dig @ns1.desec.io omarchy.az A +short # expect the four GitHub IPs
    dig omarchy.az A +short               # expect the same, via public resolvers
    curl -sI https://omarchy.az           # expect HTTP/2 200

## If it breaks

| Symptom | Likely cause |
|---|---|
| `dig omarchy.az NS` empty | Delegation lost at online.az |
| deSEC returns `REFUSED` | Zone deleted at deSEC |
| Resolves, but `404` | Custom domain or `CNAME` file changed |
| Works elsewhere, not for you | Local DNS cache. `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder` |

That last one is more common than it sounds — router, VPN, and macOS each cache
independently, and a negative (`NXDOMAIN`) answer can persist for hours.

## Local preview

    python3 -m http.server 8000

Then open <http://localhost:8000>.

## Deploy

Push to `main`. That's the whole deploy.
