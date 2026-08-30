# SyeLabs Roadmap — Firebase Setup

The app runs today in **local-storage mode**: it works, but data lives in one
browser on one machine. These four steps turn on shared cloud sync with
Google sign-in restricted to an email allowlist.

---

## 1. Paste the config into `roadmap.html`

Firebase console → **Project settings** (gear icon) → **Your apps** → the Web
app → **SDK setup and configuration** → **Config**.

If no Web app exists yet, click **Add app → Web** (`</>`), give it a nickname,
and skip Firebase Hosting.

Copy the six values into `SYELABS_FIREBASE_CONFIG` near the top of
`roadmap.html` (around line 156), replacing every `PASTE_` placeholder.

These values are **not secrets** — they ship in every Firebase web app and are
safe in a public repo. Access is controlled by step 3, not by hiding these.

## 2. Turn on Google sign-in

Firebase console → **Authentication** → **Get started** → **Sign-in method** →
**Google** → enable → set a support email → **Save**.

Then under **Authentication → Settings → Authorized domains**, confirm the
domains the page will be served from:

- `localhost` — already there by default, covers local testing
- `<username>.github.io` — add this if you publish via GitHub Pages

A domain that is missing here fails with `auth/unauthorized-domain`, which the
sign-in screen reports in plain language.

## 3. Publish the security rules — do not skip this

Firebase console → **Firestore Database** → **Create database** → start in
**production mode** → pick a region close to your team.

Then **Rules** → paste the contents of `firestore.rules` → **Publish**.

This file is the actual access control. Until it is published, Firestore's
default production rules deny everyone, and the app will show
"You don't have access" for every account — including yours.

## 4. Add your team

Edit `allowedEmails()` in `firestore.rules` and republish:

```
function allowedEmails() {
  return [
    'annushapervez7@gmail.com',
    'cochair@syelabs.com',
    'treasurer@syelabs.com'
  ];
}
```

Lowercase addresses only — the rule lowercases the incoming token before
comparing. Adding or removing someone is a rules change only; `roadmap.html`
never needs to be touched.

---

## How access works

The allowlist lives in **one place**: `firestore.rules`, enforced on Google's
servers. The page has no copy of the list and cannot be edited to grant access.

An unauthorized account signs in successfully, then its first Firestore read is
rejected with `permission-denied`, which the app surfaces as the
"You don't have access" screen and clears any locally cached data.

Sign-out also clears the local cache, so organizational data is not left behind
on a shared machine.

## Running locally

```
python3 -m http.server 8000
```

Then open <http://localhost:8000/roadmap.html>.

Opening the file directly with `file://` will not work — browsers block ES
module loads over that scheme, and Firebase Auth rejects it as an origin.

## Troubleshooting

| Symptom | Cause |
|---|---|
| Header shows "Local only" | Config still has `PASTE_` placeholders |
| Everyone sees "You don't have access" | `firestore.rules` not published yet (step 3) |
| One person sees "You don't have access" | Their address is missing or misspelled in `allowedEmails()` |
| `auth/unauthorized-domain` | Serving domain missing from Authorized domains (step 2) |
| `auth/popup-blocked` | Browser blocked the popup; allow popups for the site |
