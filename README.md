# LTP3

**Lucky & Tigrou Party 3 - "Make A Wish !"**, a French demoparty held near Paris on 27-29 August
1999 (the "true oldschool party"). This is its public website as it stood after the party: news,
information, competitions, results, balance, how to come, registration and contact pages, and the
questions and answers forum. The party's own event site (intranet) is a separate archive.

Archive of an old dynamic website (classic ASP and SSI pages, flat `.txt` files), published as a
**100 % static, read-only site** on bunny.net at the hostname of [`site.json`](site.json).
Kept online for history, no longer developed. Visitors can never write anything: the pages are
the cleaned, redacted static pages committed in `site/`, the CDN only serves files, and every
method except GET/HEAD/OPTIONS is refused at the edge.

## Layout

| Path | Content |
| --- | --- |
| `site/` | The cleaned, redacted static site, exactly as served |
| `private/` | Raw originals (git-ignored, never published): `site-raw/` (the `http/` folder), `ltpdata/forum/` (forum flat files), `secure/` (reservation pages, not used), `wayback/` (CDX listing and captures) |
| `hosting/404.html` | Custom 404 page, deployed next to `site/` |
| `site.json` | The only site-specific settings: repository, storage zone, pull zone, hostname, 404 page |
| `infra/` | OpenTofu: storage zone, pull zone, hostname, hardening edge rules (state encrypted, committed) |
| `infra/secrets.env` | Proton Pass **references** to the secrets (vault `OldSites`), no secret values |
| `scripts/statify.py` | Raw originals (`private/`) → cleaned `site/` (run locally) |
| `scripts/verify_build.py` | Checks `site/`: no ASP/SSI, forms, counters, e-mails, credentials, new broken links |
| `scripts/check_excludes.py` | Fails if `site/` holds a path of `scripts/exclude-globs.txt` |
| `scripts/preview.py` | Local preview with the charsets of the edge rules (`.shtml` shown as HTML) |
| `scripts/deploy.py` | Sync `site/` to the storage zone, purge the CDN |
| `scripts/smoke_test.py` | Check the public site: pages as in `site/`, POST refused, security headers, charset, 404 |
| `scripts/bootstrap.sh` | One-time setup (`new` for this site, `github` for the repository and secrets) |

DNS is not managed here: the hostname is a CNAME to `<pull zone>.b-cdn.net` at Infomaniak.

## What was changed from the original

Source: `Sites web/ltp3/http/` on the NAS (the site root) and `Sites web/ltp3/ltpdata/forum/`
(the forum flat files), copied into `private/`.

- **URLs kept.** `index.html` (logo page), `home.shtml`, `info.shtml`, `competition.shtml`,
  `results.shtml`, `ltp3resf.txt`, `bilan.shtml`, `tocome.shtml`, `reserve.shtml`, `contact.shtml`,
  `tshirt.html`, `img/`, `on/`: same names. The SSI include of `menu.inc` is inlined; the `.shtml`
  pages are plain HTML served as `text/html`.
- **ASP pages rendered once.** `forum.asp` (questions and answers, 98 messages of 1999-05-31 to
  1999-08-20 and 72 answers of the organizing team) -> `forum.html`, rendered from the flat files,
  newest first. Its question form is replaced by an "archived, read-only" notice; the admin
  answer/delete links are gone. `cartrips.asp` (car sharing) -> `cartrips.html`, a notice only:
  the offers were personal data. The menu links to these two pages pointed to the 1999 server by
  its bare IP address; they now use the local names. `build-meta/redirects.tsv` lists the old ->
  new names.
- **Registration**: `reserve.shtml` keeps its text, but the lists of the people who had
  registered (boys online, boys by cheque, girls, foreigners, organizers: handles and groups) are
  replaced by a notice. The live form `reserve.asp` and the `secure/` copies are not published.
- **Redacted**: the phone number of the party place (2 occurrences, `home.shtml`; the postal address
  of the Orange Juice association, for the T-shirt cheques, is kept as in the original); e-mail links of `home.shtml`,
  `reserve.shtml` and `contact.shtml` (`contact.shtml` says so on the page); e-mail domains in the
  forum messages (`nick@????`).
- **Added**: nothing beyond the notices above. `face="verdana"` gets sans-serif fallbacks
  (`DejaVu Sans, Geneva, sans-serif`) for systems without Verdana.
- **Not published** (`scripts/exclude-globs.txt`): every `.asp` source (`getadmin.asp`,
  `reserve.asp`, ...), `menu.inc` and `menuasp.inc`, `votes.bak`, `csv2sql_*.vbs`, the shell
  dotfiles of the server (`.bash*`, `.Xdefaults`, `.inputrc`) and the `ltpdata` folders `logins`
  and `carlogins` (credentials, car trips), `french.dic` and `#total.txt`. None of them was
  copied into `private/`, except the forum.
- **Left as they were**: dead external links (planet-d.net mirrors, scene.org FTP, ratp.fr, sponsors)
  and two files that were already missing in 1999: `ltp3-invitation.zip` (the invitation intro,
  linked as "local") and `img/carte2.jpg`.
- **Not part of this archive**: the event site (`ltp3/party/`, `party_org/`: another archive).

## Wayback search (2026-11-23)

Looked for LTP3-era (1999-2000) captures to fill what is missing (CDX API, `id_` URLs, 2 s
delay): `ltp.ojuice.net/` (all captures are from 2000-08 on: the LTP4 site), `www.ojuice.net/ltp*`,
`ojuice.net/ltp*`, `www.planet-d.net/ltp*`, `tgv.citeweb.net/ltp*`, `207.153.236.79/ltp*` (nothing),
`ltp.planet-d.net/` (home page 1999-10-13 and 2000-03-02, `home.shtml`, `info.shtml`,
`competition.shtml` of 1999-10-13). Those pages are earlier snapshots of the pages already in the
NAS copy (same images, same text); no image is missing, so nothing was imported.
`ltp3-invitation.zip` and `img/carte2.jpg` were never captured (a 2007 capture of the zip is a
493-byte error page). The listing is kept in `private/wayback/cdx-ltp3.txt`.

## Regenerating `site/`

`site/` is generated: never edit it by hand, fix `scripts/statify.py` and regenerate. The raw
originals live in `private/` (git-ignored; `$LTP3_RAW` and `$LTP3_FORUM` override the
paths); the build never reads the NAS. Keep `private/`, it is the working copy.

```bash
python3 scripts/statify.py && python3 scripts/verify_build.py && python3 scripts/check_excludes.py
```

```bash
python3 scripts/preview.py 3719
```

## Hardening (`infra/main.tf`)

- Edge rule 1: any method other than GET, HEAD, OPTIONS is blocked at the edge, before the
  origin (bunny.net answers 403; edge rules cannot return 405). `block_post_requests` is also on.
- Edge rule 2: `Content-Security-Policy` (same-origin, inline script/style allowed, images from
  the site only, `form-action 'none'`), `X-Content-Type-Options: nosniff`,
  `X-Frame-Options: SAMEORIGIN`, `Referrer-Policy`, HSTS, `Permissions-Policy`.
- Edge rules 3-4: `Content-Type` with `charset=iso-8859-1` for `.htm/.html/.shtml` and
  `.txt/.c` (the pages are ISO-8859-1).
- Edge rule 5: browser cache 7 days for images and archives, 1 hour for pages; CDN cache 30
  days, purged on each deployment.
- Archives (`zip swf`): `application/octet-stream` + `Content-Disposition: attachment`.
- COOP/CORP `same-origin`. Search engines are allowed (no `X-Robots-Tag`, no `robots.txt`).
- CI (deploy workflow), before upload: excluded-paths check, `scripts/verify_build.py --build site`
  and a ClamAV scan of `site/`.
- Cookies are stripped before the origin; TLS "Compatible"; forced HTTPS.
- Custom 404: `hosting/404.html` (`not_found_page` in `site.json`).

## Deploying

- **Website**: push a change under `site/` or `hosting/` to `main` → workflow *Website: deploy*
  runs the checks, uploads the changed files, deletes the removed ones, purges the CDN and
  runs the smoke test. A manual run allows a dry run.
- **Infra**: pull requests touching `infra/` show the plan; apply is a manual run of
  *Infra: bunny.net* (action `apply`, from `main`), which commits the encrypted state, deploys
  `site/` and runs the smoke test.

GitHub secrets: `BUNNY_API_KEY` (bunny.net account API key) and `TF_VAR_STATE_PASSPHRASE`
(state encryption), both from Proton Pass vault `OldSites`.

Locally (Proton Pass CLI logged in):

```bash
pass-cli run --env-file infra/secrets.env -- python3 scripts/deploy.py --dry-run
```

```bash
cd infra && pass-cli run --env-file secrets.env -- tofu plan
```

## First setup

1. Write `site.json` and validate offline (secrets are the shared ones of the other old sites):

   ```bash
   scripts/bootstrap.sh new LTP3 ltp3 ltp3 ltp3.sauthier.family
   ```

2. Create the GitHub repository and its secrets, push, open the PR:

   ```bash
   scripts/bootstrap.sh github
   ```

3. After merge, run the *Infra: bunny.net* workflow with action `apply`: it creates the zones,
   hostname and edge rules, deploys `site/` and runs the smoke test.
4. At Infomaniak (Manager > Domaines > sauthier.family > Zone DNS), add a CNAME
   `ltp3` -> `ltp3.b-cdn.net`. The certificate is issued once DNS answers; re-run the smoke test
   (*Monitoring: weekly smoke test*, manual run) after a few minutes.

## Manual steps in the bunny.net dashboard (only if the apply cannot do it)

If the apply rejects an edge rule or the 404 path, set it by hand (CDN > pull zone
`ltp3`):

1. *Edge rules* > *Add Edge Rule*: condition *Request Method* is none of `GET`, `HEAD`,
   `OPTIONS` → action *Block Request*.
2. *Edge rules* > *Add Edge Rule*: condition *URL* `*` → one *Set Response Header* action per
   header listed above (values in `infra/main.tf`, `local.csp` and the rule 2 block).
3. *Edge rules*: condition *URL Extension* `htm`, `html`, `shtml`, `SHTML` → *Set Response Header*
   `Content-Type: text/html; charset=iso-8859-1`.
4. Storage > storage zone `ltp3` > *Error pages*: set the 404 file to `/404.html`.
