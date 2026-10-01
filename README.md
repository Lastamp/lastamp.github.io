# lastamp.github.io: publishing the site

Static site for Lastamp: landing page, Privacy Policy, Support and a 404 page, in 8 languages (en, ru, uk, es, pt-BR, de, fr, ja).
**One language per page.** English stays at the URLs the app and App Store Connect use (`/`, `/privacy/`, `/support/`, also the
`x-default`); every other language has its own path: `/ru/`, `/ru/privacy/`, `/ru/support/`, … (`/uk/`, `/es/`, `/pt-br/`, `/de/`, `/fr/`,
`/ja/`). Every page links all 8 versions with `hreflang` + `x-default`; `sitemap.xml` lists every URL and `robots.txt` points to it.
The language switcher is a set of ordinary links. A tiny inline script remembers an explicit choice (`localStorage` `lastamp.lang`);
on the English pages only, old links like `/#ru` or `/privacy/#ru_battery` go to the matching path, and on the first page of a visit the
saved choice (or else the browser language) is opened once. Language pages never redirect. Without JavaScript everything works via links.
404 is English only.

The pages are generated. Edit the text in `docs/tools/site_src/<lang>.py` (the Lastamp repo), then run
`python3 docs/tools/site_src/build.py` and check with `python3 docs/tools/site_check.py`. Don't hand-edit `site/**/*.html`,
`sitemap.xml` or `robots.txt`, because the next build overwrites them.

**Static assets** (kept in `site/`, not written by `build.py`):
- `site/fonts/` — self-hosted fonts, no Google Fonts: Tektur 700/900, IBM Plex Sans 400/600, JetBrains Mono 500/700, WOFF2,
  subsets latin + latin-ext + cyrillic (Japanese uses system fonts), from `@fontsource/*` 5.3.0 (SIL Open Font License; the licence
  texts `OFL-*.txt` must stay next to the fonts). The version is in the file name (`tektur-latin-900.v5.3.0.woff2`) so a new
  version never hits an old cache. To update: `npm pack @fontsource/tektur @fontsource/ibm-plex-sans @fontsource/jetbrains-mono`,
  copy `files/<family>-<subset>-<weight>-normal.woff2` as `<family>-<subset>-<weight>.v<version>.woff2` + `LICENSE` as `OFL-<Family>.txt`,
  set `FONT_VER` (and weights, if the CSS changed) in `build.py`, rebuild. `build.py` stops if a font file is missing.
- `site/og.png` — the 1200×630 link-preview picture (`og:image`), made by `python3 docs/tools/site_src/og_image.py` (Pillow) from the
  app icon and the English tagline. Re-run it when the icon or the tagline changes.
- `site/app-ads.txt`, `site/.nojekyll` — see below.

## 1. Contact and developer

Contact: **lastamp.app@gmail.com** · developer: **Artem** — set in `docs/tools/site_src/build.py` (`CONTACT`, `DEVELOPER`).
The developer name should match the seller name shown in App Store Connect; if it differs, change `DEVELOPER` and rebuild.

## 2. Create the GitHub repo

1. Sign in to GitHub. To get `https://lastamp.github.io/`, the **account or organisation must be named `lastamp`**:
   create a free organisation `lastamp` (github.com → + → New organization), or use your own account. The site will then live at
   `https://<account>.github.io/`, and every URL below changes to match.
2. Create a **public** repository named **exactly** `lastamp.github.io` (`<account>.github.io` for another account).
3. Push the **contents** of `site/` (not the folder itself) to the `main` branch — **all of it**: the HTML in every language folder
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
Wait about 1 minute, then open https://lastamp.github.io/privacy/ and https://lastamp.github.io/ru/privacy/, check that
`https://lastamp.github.io/privacy/#ru` lands on `/ru/privacy/`, that https://lastamp.github.io/sitemap.xml and `/robots.txt` open, and that
a wrong URL gives the styled 404. Optionally submit the sitemap in Google Search Console.
Tick **Enforce HTTPS** if it is shown.

## 4. Wire it into the app and the store

- `Config/Lastamp-Info.plist` → `LastampPrivacyPolicyURL` = `https://lastamp.github.io/privacy/`
  (the in-app menu link uses it; put the same URL into the AdMob GDPR message: AdMob → Privacy & messaging).
- `site/app-ads.txt` (served at `https://lastamp.github.io/app-ads.txt`): AdMob checks it on the developer website listed
  in the App Store. Replace `pub-XXXXXXXXXXXXXXXX` with your AdMob publisher ID (AdMob → Settings → Account information),
  keep `f08c47fec0942fa0` (Google's certification ID). The file must stay at the site root, not in a subfolder.
  `docs/tools/site_src/build.py` does not generate or delete it.
- App Store Connect → App Information → **Privacy Policy URL** = the same URL; **Support URL** = `https://lastamp.github.io/support/`;
  Marketing URL (optional) = `https://lastamp.github.io/`.
- App Privacy answers: `docs/store/app-privacy.md`.

## Updating

Change the text in the generator → build → `site_check.py` → replace placeholders → push everything in `site/`
(GitHub Pages caches for about 10 minutes). Update the effective date (the `effective` string in each
language file) whenever the meaning of the policy changes.
