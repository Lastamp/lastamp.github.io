# lastamp.github.io: publishing the site

Static site for Lastamp: landing page (`/`), Privacy Policy (`/privacy/`), Support (`/support/`) and a 404 page.
Each page contains all 8 languages (en, ru, uk, es, pt-BR, de, fr, ja). The language is picked from the URL hash (`#ru`), then from the
saved choice, then from the browser language. Without JavaScript the page shows English.

The pages are generated. Edit the text in `docs/tools/site_src/<lang>.py` (the Lastamp repo), then run
`python3 docs/tools/site_src/build.py`. Don't hand-edit `site/*.html`, because the next build overwrites it.

## 1. Contact and developer

Contact: **lastamp.app@gmail.com** · developer: **Artem** — set in `docs/tools/site_src/build.py` (`CONTACT`, `DEVELOPER`).
The developer name should match the seller name shown in App Store Connect; if it differs, change `DEVELOPER` and rebuild.

## 2. Create the GitHub repo

1. Sign in to GitHub. To get `https://lastamp.github.io/`, the **account or organisation must be named `lastamp`**:
   create a free organisation `lastamp` (github.com → + → New organization), or use your own account. The site will then live at
   `https://<account>.github.io/`, and every URL below changes to match.
2. Create a **public** repository named **exactly** `lastamp.github.io` (`<account>.github.io` for another account).
3. Push the **contents** of `site/` (not the folder itself) to the `main` branch:

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
Wait about 1 minute, then open https://lastamp.github.io/privacy/ and check `#ru`, `#ja` and a wrong URL (you should get the styled 404).
Tick **Enforce HTTPS** if it is shown.

## 4. Wire it into the app and the store

- `Config/Lastamp-Info.plist` → `LastampPrivacyPolicyURL` = `https://lastamp.github.io/privacy/`
  (the in-app menu link and the AppLovin consent flow both use it).
- App Store Connect → App Information → **Privacy Policy URL** = the same URL; **Support URL** = `https://lastamp.github.io/support/`;
  Marketing URL (optional) = `https://lastamp.github.io/`.
- App Privacy answers: `docs/store/app-privacy.md`.

## Updating

Change the text in the generator → build → replace placeholders → push. Update the effective date (the `effective` string in each
language file) whenever the meaning of the policy changes.
