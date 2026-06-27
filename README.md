# myfitnessBro

A quick macro tracker you install on your phone. Type something like **"200g grilled potato"** or scan a barcode — it pulls the macros from a bundled food database, fills the calorie ring, and saves the food for next time.

Built the same way as **liftlog**: single HTML file, hosted on GitHub Pages, installed on iPhone via "Add to Home Screen". **No API keys, no backend, no setup for friends.** Each person's data lives only on their own phone.

## What's in this folder

```
myfitnessbro/
├── index.html              ← the whole app
├── foods.json              ← bundled food database (~370 curated foods)
├── manifest.webmanifest    ← PWA metadata
├── sw.js                   ← service worker (offline cache)
├── icons/
│   └── icon.svg            ← (PNG icons go here too — see Part 0)
└── README.md
```

No `worker/`, no Cloudflare, no Gemini, no Claude, no keys. Everything runs on your phone.

---

## How food logging works

1. **Type something in the bar** — e.g. `"200g grilled potato"`, `"two eggs"`, `"1 banana"`, `"shawarma chicken 200g"`.
2. **App checks "My Foods" first** (your saved foods). If found → instant add.
3. **Otherwise it searches the bundled database** (~370 foods) for matches.
4. **If multiple matches** → bottom-sheet picker shows the top 5 with macros scaled to your quantity.
5. **One match or you picked one** → confirm card → ↑ Add to log.
6. **The food auto-saves to My Foods**, so re-typing later skips the picker entirely.

If nothing in the database matches, a **manual quick-add form** appears (name + qty + kcal + P/C/F). You type it once and it's in your library forever.

### Barcode scanner

The ||| button opens the camera. Point at any product barcode. The app queries **[Open Food Facts](https://world.openfoodfacts.org)** (free, ~3M products, no key) and prefills the confirm card with the product's per-100g macros.

> Requires **iOS 17+** for the `BarcodeDetector` API. On older iOS, barcode won't work; text + manual still does.

---

## Part 0 — One-time icon prep (optional, do later)

The app ships with an SVG icon. iOS prefers PNGs at specific sizes for the home-screen tile. You can:

- Skip this for now (the SVG works fine in the browser; the install icon will be a default), **or**
- Open `icons/icon.svg` in any image editor and export PNGs named:
  - `icons/icon-192.png`        (192×192)
  - `icons/icon-512.png`        (512×512)
  - `icons/icon-maskable-512.png` (512×512, with ~10% padding on all sides)
  - `icons/apple-touch-icon-180.png` (180×180, the iPhone home-screen icon)

The service worker tolerates these being missing during dev.

---

## Part 1 — Publish to GitHub Pages

This puts the app online at a URL you can install on any phone.

### 1. Create a new repository

1. Go to **[github.com/new](https://github.com/new)**.
2. **Repository name:** `myfitnessbro` (lowercase, no spaces).
3. **Public.** (Pages requires public on a free account.)
4. Leave everything else as-is and click **Create repository**.

### 2. Upload the files

1. On the new empty repo page, click **uploading an existing file**.
2. Drag the entire contents of your `myfitnessbro` folder — `index.html`, `foods.json`, `manifest.webmanifest`, `sw.js`, and the `icons` folder. **Don't drag the `myfitnessbro` folder itself; drag what's inside it.**
3. Scroll down and click **Commit changes**.

### 3. Turn on GitHub Pages

1. In the repo, click **Settings** (top right of the repo nav).
2. Left sidebar → **Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Under **Branch**, pick `main` and `/ (root)`. Click **Save**.
5. Wait ~1 minute. Refresh the Pages settings page. You'll see:
   > Your site is live at `https://<yourusername>.github.io/myfitnessbro/`

That URL is your app.

---

## Part 2 — Install on iPhone

1. Open **Safari** on the iPhone (must be Safari — Chrome on iOS can't install PWAs).
2. Go to your URL.
3. Tap **Share** → scroll down → **Add to Home Screen**.
4. Confirm the name and tap **Add**.

The app opens full-screen with no Safari address bar. Works **offline from this point on** (service worker caches everything on first load).

### Share with friends

Just send them the URL. They follow the same Add-to-Home-Screen steps. Zero setup — no accounts, no keys, no nothing. Each person's data stays only on their own phone.

---

## Part 3 — Using the app

- **Magic input bar:** type freely. `"200g grilled potato"`, `"two scrambled eggs"`, `"1 banana"`, `"shawarma chicken 200g"`.
- **Picker (multiple matches):** tap the one you meant. After that, re-typing it is instant from My Foods.
- **Manual add:** if nothing matches, the form opens prefilled with whatever you typed. Fill kcal + P/C/F once, it's saved forever.
- **Barcode (||| icon):** point at a barcode. Open Food Facts looks it up.
- **Tap any logged entry** to adjust the quantity or delete it.
- **Foods tab:** browse your saved foods. Tap to log, trash icon to delete.
- **Goals tab:** set daily kcal + protein/carbs/fat targets.
- **More tab → Export backup / Import backup:** JSON file, same pattern as liftlog. Export every now and then to iCloud Drive or email.

### Why backups matter

Data lives in browser storage on this phone. It's **not** synced and **not** transferred when you switch phones. Export a JSON now and then.

---

## Part 4 — Extending the food database

`foods.json` is editable plain JSON. Each entry:

```json
{
  "name": "Grilled potato",
  "aliases": "potato grilled",
  "category": "Vegetables",
  "unit": "g",
  "per": 100,
  "kcal": 93,
  "p": 2.5, "c": 21, "f": 0.1,
  "fiber": 2.2, "sugar": 1.2, "sodium": 10
}
```

- `name` — canonical display name. Shown in the picker and in My Foods.
- `aliases` — extra search tokens. Help the matcher find this entry from different phrasings (e.g. plural, reorder, casual names).
- `category` — used to group by category in the picker subtitle.
- `unit` — `"g"`, `"ml"`, `"piece"`, or `"serving"`.
- `per` — the quantity the macros below correspond to. `kcal`/`p`/`c`/`f`/`fiber`/`sugar`/`sodium` are scaled proportionally.
- `fiber`/`sugar` in grams, `sodium` in milligrams.

After editing `foods.json`, bump `CACHE_VERSION` in `sw.js` to the next number so phones pick up the new file.

### Sources for macros

When adding to the curated set, get numbers from authoritative sources:

- **[USDA FoodData Central](https://fdc.nal.usda.gov)** — gold standard for generic foods (English).
- **[CIQUAL](https://ciqual.anses.fr)** — French food agency, great for European/Mediterranean foods.
- **The brand's own nutrition label** — for packaged products.

---

## Part 5 — Updating the app

When you change any file:

1. Open `sw.js` and bump: `var CACHE_VERSION = "myfitnessbro-v8";` (then `v9`, `v10`, …).
2. Upload the changed files to the repo.
3. On each phone, **fully close the app and reopen it**. The new service worker installs and replaces the old cache.

If you forget to bump, phones keep serving the cached version forever.

---

## Tech notes

- **Offline:** everything (UI + food DB + your log + library) works offline once the app's cached. The only thing that needs network is the barcode scanner (Open Food Facts).
- **Storage:** localStorage for state, IndexedDB available if the dataset ever outgrows it. Service worker handles the cache layer.
- **Search:** token-based matching with stopword + plural handling. ~370 foods searched in <1ms in pure JS — would handle 10× this comfortably.
- **No dependencies, no build step.** Edit `index.html`, refresh.

---

## Local dev

If you want to edit `index.html` and preview before pushing to GitHub:

```bash
# any tiny static server. From the myfitnessbro/ folder:
npx http-server -p 5174 -c-1
# then open http://localhost:5174
```

Or just open `index.html` directly — most of the app works as a `file://` page, but barcode scanning needs HTTPS or localhost.
