# lastamp.app: publishing the site

> **Since 03.10.2026 the site lives on its own domain https://lastamp.app.** The repo is still `Lastamp/lastamp.github.io`
> on GitHub Pages; `site/CNAME` (from `web/public/CNAME`) = `lastamp.app`. DNS at the registrar: apex `A` → 185.199.108.153 /
> .109.153 / .110.153 / .111.153 (`AAAA` 2606:50c0:8000::153 … 8003::153), `www` `CNAME` → `lastamp.github.io`. GitHub → repo
> Settings → Pages → Custom domain = `lastamp.app`, then **Enforce HTTPS**. The old lastamp.github.io redirects automatically.

Static site for Lastamp: landing page, Privacy Policy, Support and a 404 page, in 8 languages (en, ru, uk, es, pt-BR, de, fr, ja).
**One language per page.** English stays at the URLs the app and App Store Connect use (`/`, `/privacy/`, `/support/`, also the
`x-default`); every other language has its own path: `/ru/`, `/ru/privacy/`, `/ru/support/`, … (`/uk/`, `/es/`, `/pt-br/`, `/de/`, `/fr/`,
`/ja/`). Every page links all 8 versions with `hreflang` + `x-default`; `sitemap.xml` lists every URL and `robots.txt` points to it.
The language menu («EN ▾») is a list of ordinary links (works without JavaScript). A tiny inline script remembers an explicit choice
(`localStorage` `lastamp.lang`); on the English pages only, old links like `/#ru` or `/privacy/#ru_battery` go to the matching path, and on
the first page of a visit the saved choice (or else the browser language) is opened once. Language pages never redirect. 404 is English only.

**The site is built with Astro** from `web/` in the Lastamp repo (the landing «Разряд», Support, Privacy, 404, `sitemap.xml`,
`robots.txt`). Texts: `web/src/i18n/strings.json` (landing, support, menu, footer — 8 languages, same keys; owner: content-designer) and
`web/src/i18n/privacy/<lang>.ts` (the Privacy Policy). Build and check:

```bash
cd web && npm ci && npm run build      # writes the complete site/ (everything except this README)
cd .. && python3 docs/tools/site_check.py
```

Don't hand-edit anything in `site/` except this README: the next build replaces it (`site_check.py` compares `site/` with a fresh build).

**Static assets** live in `web/public/` and are copied to `site/` by the build:
- `fonts/` — self-hosted fonts, no Google Fonts: Tektur 700/900, IBM Plex Sans 400/600, JetBrains Mono 500/700, WOFF2,
  subsets latin + latin-ext + cyrillic (Japanese uses system fonts), from `@fontsource/*` 5.3.0 (SIL Open Font License; the licence
  texts `OFL-*.txt` must stay next to the fonts). The version is in the file name (`tektur-latin-900.v5.3.0.woff2`) so a new
  version never hits an old cache. To update: `npm pack @fontsource/tektur @fontsource/ibm-plex-sans @fontsource/jetbrains-mono`,
  copy `files/<family>-<subset>-<weight>-normal.woff2` as `<family>-<subset>-<weight>.v<version>.woff2` + `LICENSE` as `OFL-<Family>.txt`,
  set `FONT_VER` (and weights, if the CSS changed) in `web/src/site.ts` and `web/scripts/og_image.py`, rebuild. The build stops if a font file is missing.
- `og.png` — the 1200×630 link-preview picture (`og:image`), made by `python3 web/scripts/og_image.py` (Pillow) from the
  app icon and the English tagline (`site.tagline`). Re-run it when the icon or the tagline changes, then rebuild.
- `app-ads.txt`, `.nojekyll` — see below.

## 1. Contact and developer

Contact: **lastamp.app@gmail.com** — set in `web/src/i18n/index.ts` (`CONTACT`); footer «© {year} Lastamp Team» — `web/src/layouts/Base.astro`.
Privacy/Terms name the developer as «the Lastamp team» (texts in `web/src/i18n/privacy/`, `terms/`); keep it consistent with the seller name in App Store Connect.

## 2. Create the GitHub repo

1. Sign in to GitHub. To get `https://lastamp.github.io/` (now redirecting to lastamp.app), the **account or organisation must be named `lastamp`**:
   create a free organisation `lastamp` (github.com → + → New organization), or use your own account. The site will then live at
   `https://<account>.github.io/`, and every URL below changes to match.
2. Create a **public** repository named **exactly** `lastamp.github.io` (`<account>.github.io` for another account).
3. Build (`cd web && npm ci && npm run build`), then push the **contents** of `site/` (not the folder itself) to the `main` branch — **all of it**: the HTML in every language folder
   (`ru/`, `uk/`, `es/`, `pt-br/`, `de/`, `fr/`, `ja/`), `fonts/` (WOFF2 + OFL licences), `og.png`, `sitemap.xml`, `robots.txt`,
   `404.html`, `app-ads.txt` and `.nojekyll`. Without `fonts/` the pages fall back to system fonts; without the language folders the
   hreflang links point to 404s.

```bash
cd site
git init -b main
git add -A            # includes .nojekyll
git commit -m "Lastamp site"
git remote add origin https://github.com/lastamp/lastamp.github.io.git
git push -u origin main
```

## 3. Turn on Pages

Repo → **Settings → Pages** → Source: **Deploy from a branch** → Branch: **main**, folder **/ (root)** → Save.
Wait about 1 minute, then open https://lastamp.app/privacy/ and https://lastamp.app/ru/privacy/, check that
`https://lastamp.app/privacy/#ru` lands on `/ru/privacy/`, that https://lastamp.app/sitemap.xml and `/robots.txt` open, and that
a wrong URL gives the styled 404. Optionally submit the sitemap in Google Search Console.
Tick **Enforce HTTPS** if it is shown.

## 4. Wire it into the app and the store

- `Config/Lastamp-Info.plist` → `LastampPrivacyPolicyURL` = `https://lastamp.app/privacy/`
  (the in-app menu link uses it; put the same URL into the AdMob GDPR message: AdMob → Privacy & messaging).
- `web/public/app-ads.txt` → `site/app-ads.txt` (served at `https://lastamp.app/app-ads.txt`): AdMob checks it on the developer website listed
  in the App Store. Replace `pub-XXXXXXXXXXXXXXXX` with your AdMob publisher ID (AdMob → Settings → Account information),
  keep `f08c47fec0942fa0` (Google's certification ID). The file must stay at the site root, not in a subfolder.
  The build copies it unchanged.
- App Store Connect → App Information → **Privacy Policy URL** = the same URL; **Support URL** = `https://lastamp.app/support/`;
  Marketing URL (optional) = `https://lastamp.app/`.
- App Privacy answers: `docs/store/app-privacy.md`.

## Updating

Change the source in `web/` → `cd web && npm ci && npm run build` → `python3 docs/tools/site_check.py` (0 FAIL) → copy the contents
of `site/` into the `lastamp.github.io` repo (local copy: the `lastamp-site` folder), commit and push (GitHub Pages caches for about
10 minutes). Update the effective date (`effective` in each `web/src/i18n/privacy/<lang>.ts`) whenever the meaning of the policy changes.
