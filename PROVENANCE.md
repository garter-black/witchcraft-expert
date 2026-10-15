# witchcraft-expert — provenance

Cloudflare Pages project `witchcraft-expert`, account `86cf181f919e67fc61e33a0d578903b0`.
Public host `witchcraft.expert`; Pages host `witchcraft-expert.pages.dev`.
Established by the estate source-of-truth lane on **2026-10-16**. Nothing was deployed.

## Project-to-directory mapping — how it was confirmed, not guessed

| Path | Confirmation |
|---|---|
| `sites/witchcraft-expert/restore/` | `restore/.wrangler/cache/pages.json` → `"project_name": "witchcraft-expert"` |
| `sites/witchcraft-expert/build-a-brain/` | `build-a-brain/.wrangler/cache/pages.json` → `"witchcraft-expert"` |
| `artifacts/EXPERT-PUBLIC-CURATION-20260921/before-worker.mjs` | sha256 equals `witchcraft-expert-route-repair.worker.mjs` in `policies/witchcraft-identity.json` |

## What lives here

```
pages/index.html                                  the Pages homepage (canonical site content)
pages/build-a-brain/index.html                    /build-a-brain/
pages/robots.txt  pages/sitemap.xml               crawler config
worker/witchcraft-expert-route-repair.worker.mjs  the ONLY reviewed-baseline Worker copy found on disk
undeployed-route-repair/worker.mjs                a later local variant that is NOT live (see below)
```

## Two Workers, one identity — read this before deploying

`policies/witchcraft-identity.json` pins `witchcraft-expert-route-repair` to
`worker.mjs = c43aeb7594b67dcad8cf5f928f5ec02ab3ceeae8bc461bee7a66e5b8787dcc21`
(baseline version `50a72e1e-3287-40f4-ab03-f95ddaf1fcfe`, and active == baseline).

Exactly **one** file on this disk matches that reviewed hash:
`artifacts/EXPERT-PUBLIC-CURATION-20260921/before-worker.mjs`. It is what is committed
as the Worker source, and it reproduces the live `/services` page **byte-for-byte**
after stripping Cloudflare's identity-footer injection.

`sites/witchcraft-expert/route-repair/worker.mjs` is a **different, later variant**
(sha256 `176c2d8a886fd122…`). It differs in exactly three places:
1. an added `/1000things` route and `/livingthoughts/sources`,
2. changed Living Thoughts card wording on the homepage,
3. a changed regex guard in the route matcher.

Its embedded `/services` is byte-identical to the reviewed copy, so it is not
inconsistent — it is simply **not the reviewed baseline and not deployed to that
Worker**. `witchcraft.expert/1000things` currently returns the homepage bytes with
a 200, i.e. it is served by the *Pages* origin, not by this Worker's `/1000things`
route. It is kept under `undeployed-route-repair/` so the difference stays reviewable
instead of being silently merged into the source of truth.

## Media and secrets

Excluded by `.gitignore`: all PDFs, `dist/`, `node_modules/`, `.wrangler/`,
`.dev.vars*`, `.env*`, `media/`, `assets/`.
Verified: no media blob, no key-shaped literal, and zero occurrences of
the forbidden domain in any committed blob.

## Not verified

- Nothing deployed; Cloudflare read-only throughout.
- `pages/book/SAMPLE-10p.pdf` and `restore/book/` are **not** committed (PDF by rule)
  and `/book/` returns 302 on both hosts; the redirect target was not followed.
- `/livingthoughts` returns **410 Gone** on the custom domain.
- Soft-404 behaviour: `/clone/`, `/restore/` and `/100-things.html` all return
  **200 with the homepage bytes**. This is the Pages origin's SPA-style fallback,
  not broken routing, but it means the homepage's real route list is the authority.
