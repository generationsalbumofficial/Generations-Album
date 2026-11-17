# Generation Album

A simple family-tree album for the web. Add a father and mother, then their sons and daughters with photos, and link every generation to the next one. Open any generation with one tap, share it with family, or keep it private.

The website is a small **installable web app (PWA)**: `index.html` plus a manifest, a service worker and the icons. It is hosted on GitHub Pages and can be installed from the Chrome menu. The data is stored in **your own Google Sheet** and the photos in **your own Google Drive folder**, through a small **Google Apps Script** backend. No server and no paid service is needed.

---

## Features

- **Accounts:** sign up and log in with email and password. A forgot-password mail with a temporary password is supported.
- **Generations:** each generation has a Father and a Mother. Under them you add Sons and Daughters, each with a name, a photo and a married or unmarried status.
- **Linked generations:** a married son or daughter (and the father or mother) gets a **+** button until their own family generation is added. After that it becomes an **arrow** button that opens it.
- **Photos in Google Drive:** photos are compressed in the browser and saved in your Drive folder. Nothing is stored on a third-party image host.
- **Hide a photo:** any person's photo can be hidden. A hidden photo is never sent to viewers or to the public, only to the owner and to admin-mode people.
- **Public or Private:** each generation can be made public (anyone who has its link can open it) or private.
- **Sharing with two modes:**
  - **View Mode:** can only look at the generation.
  - **Admin Mode:** can edit and update that one generation and can add the family generation of a married person. The new generation belongs to that person. Admin Mode cannot transfer, share, change Private/Public or delete the generation.
  - The owner can add people, switch their mode, or remove them at any time.
- **Transfer:** the owner can transfer a generation (and the generations below it) to another account after confirming the password.
- **Autosave:** changes are saved automatically. If the same family is edited from two phones, the site warns about the conflict instead of overwriting.
- **Languages:** the site is English by default. The menu has a **Language** option (and guests get a small language button on the first page) with 17 more languages: Hindi, Bengali, Marathi, Telugu, Tamil, Urdu, Gujarati, Kannada, Malayalam, Punjabi, Spanish, French, Arabic, Portuguese, Indonesian, Russian and Chinese. Urdu and Arabic switch the text to right-to-left. The choice is remembered on the phone.
- **Fast start:** the site remembers the logged-in user, so the home page loads with one server call.
- **Demo mode:** if `API_URL` is empty, the site runs with data stored only in the browser, which is useful for trying it out.

## How it works

```
Browser (index.html on GitHub Pages)
        │  fetch (POST, JSON)
        ▼
Google Apps Script web app  ──►  Google Sheet  (Users, Sessions, Generations, GenShares)
        │
        └──────────────────────►  Google Drive  (photo files)
```

### Sheet tabs

| Tab | Columns |
|---|---|
| `Users` | UserID, Name, Email, PasswordSalt, PasswordHash, Created, LastLogin |
| `Sessions` | TokenHash, UserID, Expires |
| `Generations` | Email, GID, Names, Images, Convertor, Marriage, LinkedGID, Updated, Visibility, Hide |
| `GenShares` | GID, Viewers |

One row is one generation. The list columns are comma separated and in the same order (Father, Mother, Son, Daughter…). `Email` is the admin (owner) of the generation.

`Viewers` stores a list of pairs, for example:

```
"person1@gmail.com,V Mod","person2@gmail.com,E Mod"
```

`V Mod` is View Mode and `E Mod` is Admin Mode.

Passwords are stored only as a salted hash. Session tokens are stored only as a hash.

## Files

| File | What it is |
|---|---|
| `index.html` | The complete website (HTML, CSS and JavaScript in one file) |
| `manifest.webmanifest` | App name, colours and icons, so Chrome can install the site as an app |
| `sw.js` | Service worker. Keeps the app shell so the app opens fast. It never caches your data or photos |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png`, `favicon-32.png` | App icons (all in the root, no icons folder) |
| `favicon.ico`, `favicon.svg` | Browser tab icons |
| `.nojekyll` | Tells GitHub Pages to publish every file as it is |
| `backend/Generation_gs.txt` | The Google Apps Script backend. Paste it into the Apps Script editor. It does not need to be hosted |
| `README.md` | This file |

## Install as an app

The site works as an app on a phone, tablet or computer. It needs to be opened over **https** (GitHub Pages does this).

**Android (Chrome):** open the site, tap the **⋮ menu**, then **Install app** (or **Add to Home screen**). The app icon appears on the home screen and opens full screen, without the address bar.

**iPhone (Safari):** tap **Share**, then **Add to Home Screen**.

**Computer (Chrome / Edge):** click the **install icon** at the right end of the address bar, or open the **⋮ menu** and choose **Install Generation Album**.

If the install option does not show, open the site once, wait a few seconds and open the menu again. After you publish a new version, the installed app updates itself the next time it is opened (change `VERSION` in `sw.js` so the old copy is replaced).

## Setup

### 1. Backend (Google Sheet + Apps Script)

1. Create a new Google Sheet.
2. Open **Extensions → Apps Script**, delete the old code and paste the full content of `backend/Generation_gs.txt`. Save.
3. Create a folder in Google Drive for the photos and share it as **Anyone with the link → Viewer**. Copy the folder ID from its address (the text after `/folders/`).
4. In the script, set `DRIVE_FOLDER_ID` to your folder ID. The folder must belong to the same Google account as the sheet.
5. In the editor run these functions once and click **Allow** each time:
   - `setup` (creates the tabs and headers)
   - `authorizeDrive`
   - `authorizeMail`
6. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
7. Copy the web app URL that ends with `/exec`.

> After **any** change in the script you must go to **Deploy → Manage deployments → Edit → Version: New version → Deploy**. Without this the old code keeps running.

### 2. Website (GitHub Pages)

1. Open `index.html` and find this line:

   ```js
   const CONFIG = { API_URL: '...' };
   ```

   Put your web app URL (the `/exec` one) between the quotes.
2. Upload **all files of the zip** (everything except the `backend` folder, which is only for Apps Script) to the root of your repository. `index.html` must stay in the root, next to `manifest.webmanifest`, `sw.js` and the icon files.
3. Go to **Settings → Pages**, choose the branch (`main`) and the folder `/ (root)`, and save.
4. After a minute the site opens at `https://USERNAME.github.io/REPO/`.

### 3. First test

Sign up with a test account, create a generation, add a person with a photo, then share it with a second account in both modes.

## Updating from the old version (the one with UID login)

Do this in order:

1. Paste the new `backend/Generation_gs.txt` into Apps Script and save.
2. Run `removeUidColumns` once. It deletes the UID columns and keeps every hidden photo hidden.
3. Run `convertGenSharesToModes` once. It converts the old share lists into the new format. Old entries become View Mode.
4. Deploy a **New version**.

## Notes and limits

- The first request after the script has been idle can take a few seconds. This is Google's cold start of Apps Script and not a problem in the site.
- Anyone who has the **link of a Drive photo** can open it. Hidden photos are not shown by the site, but if you need strong secrecy for photos, keep the Drive folder private and do not make the photos public.
- Public generations can be opened only by someone who has its Generation ID or the link. There is no search.
- Free Google accounts have daily quotas for Apps Script (emails, runtime and URL fetches). A family-sized album stays far below them.
- This project is not tested against a large number of users.

## Tech

- HTML, CSS and plain JavaScript (no framework, no build step), installable as a PWA
- Google Apps Script (backend), Google Sheets (database), Google Drive (photos)
- Fonts: DM Sans and Fraunces (Google Fonts)

## License

Add the license you want here (for example MIT), or keep the project private for family use.
