# Redcliff — Firebase + Vercel Setup Guide

Your menu is now powered by **Firebase Firestore** (real-time cloud database) and hosted on **Vercel** (free, fast, global CDN).

---

## How it works

```
Chef edits admin.html → clicks Save → data goes to Firebase Firestore (cloud)
                                              ↓
Customer opens index.html → Firebase sends live data → menu updates instantly
```

No server to manage. No cost for normal restaurant traffic. Works from any device.

---

## STEP 1 — Create a Firebase Project (5 minutes)

1. Go to **[console.firebase.google.com](https://console.firebase.google.com)**
2. Click **"Add project"** → name it `redcliff-menu` → Continue
3. Disable Google Analytics (not needed) → **Create project**

### Create the database
4. In the left sidebar → **Build → Firestore Database**
5. Click **"Create database"**
6. Choose **"Start in test mode"** (we'll secure it later) → Next
7. Pick region: **`europe-west1`** (closest to Nairobi) → Enable

### Get your config
8. Click the ⚙️ gear icon → **Project Settings**
9. Scroll to **"Your apps"** → click the **`</>`** (Web) icon
10. Register app name: `redcliff-web` → **Register app**
11. You'll see a block like this — **copy it**:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "redcliff-menu.firebaseapp.com",
  projectId: "redcliff-menu",
  storageBucket: "redcliff-menu.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef"
};
```

---

## STEP 2 — Paste your config into both files

Open **`index.html`** and **`admin.html`** in VS Code.

In each file, find the section that says:
```javascript
// ⬇⬇  PASTE YOUR FIREBASE CONFIG HERE  ⬇⬇
const firebaseConfig = {
  apiKey:            "PASTE_YOUR_API_KEY",
  ...
};
```

Replace the entire `firebaseConfig` object with the one you copied from Firebase.

**Do this in BOTH files.**

---

## STEP 3 — Secure your database (important!)

By default Firebase allows anyone to read/write. Set proper rules:

1. Firebase Console → **Firestore Database → Rules**
2. Replace the contents with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Anyone can READ the menu (customers)
    match /menu/{doc} {
      allow read: if true;
      // Only requests with the right secret can WRITE
      allow write: if request.auth != null || request.resource.data.keys().hasOnly(['sections','updatedAt']);
    }
  }
}
```

> **Simple version for now:** keep test mode (allows all reads and writes) — it works fine, and you can tighten rules later once you're up and running. Test mode expires after 30 days, so set a reminder.

---

## STEP 4 — Deploy to Vercel (3 minutes)

### Option A — Via GitHub (recommended)
1. Push your files to a GitHub repo (see README from previous version)
2. Go to **[vercel.com](https://vercel.com)** → Sign up with GitHub
3. Click **"Add New Project"** → import your `redcliff-menu` repo
4. Click **Deploy** — done!
5. Vercel gives you a URL like: `https://redcliff-menu.vercel.app`

Every time you push to GitHub, Vercel auto-deploys in ~30 seconds.

### Option B — Via Vercel CLI in VS Code
```bash
npm install -g vercel
vercel login
cd your-redcliff-folder
vercel
```
Follow the prompts. Your site goes live immediately.

---

## STEP 5 — Custom domain (optional)

1. Vercel Dashboard → your project → **Settings → Domains**
2. Add your domain (e.g. `menu.redcliff.co.ke`)
3. Point your DNS to Vercel (they give you the records)
4. Free SSL is automatic

---

## Chef Login Details

| Field    | Value          |
|----------|----------------|
| URL      | `your-site.vercel.app/admin.html` |
| Username | `chef`         |
| Password | `redcliff2024` |

**To change the password:** open `admin.html` in VS Code, find:
```javascript
const CREDENTIALS = { username: 'chef', password: 'redcliff2024' };
```
Change it, save, and push to GitHub. Vercel redeploys automatically.

---

## Daily workflow for the chef

1. Open `your-site.vercel.app/admin.html` on any phone, tablet, or laptop
2. Log in
3. Update the buffet items, change prices, add/remove dishes
4. Click **Save & Publish**
5. Every customer immediately sees the new menu — no refresh needed

---

## Costs

| Service  | Cost |
|----------|------|
| Firebase Firestore | Free up to 50,000 reads/day — more than enough |
| Vercel hosting | Free for personal/small projects |
| Custom domain | ~KES 1,500/year if you want one |

**Total running cost: KES 0** until you get very high traffic.
