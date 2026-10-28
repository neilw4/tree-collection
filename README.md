# Christmas tree collection map

One-page web app: add addresses for tree collection, mark them collected, drop piles, mark piles picked up. Everything shows live on a shared map.

- **Orange circles** = addresses waiting for collection (green + faded when collected)
- **Blue squares** = piles waiting for the van (grey + faded when picked up)
- Number on each pin = number of trees
- Tap a pin to mark done, undo, edit tree count, get directions, or delete
- The **Show** checkboxes at the top hide/show addresses, piles, and collected items (to declutter the map)
- The base map is deliberately greyscale so the coloured pins stand out (one CSS line in `index.html` if you'd like colour back)

## Try it first (no setup)

Open `index.html` in a browser. With no Firebase config it runs in **demo mode** (data saved only in that browser). Note that GPS only works once the page is served over HTTPS, i.e. after you deploy. Map tiles from a local file need a free CARTO key (see Notes at the bottom).

## Setup (about 15 minutes, free, no card needed)

### 1. Create the database
1. Go to https://console.firebase.google.com and **Add project**. Pick a hard-to-guess name (it becomes part of the URL). Turn Google Analytics off.
2. **Build → Firestore Database → Create database**. Choose **production mode** and a European location (e.g. `eur3` or `europe-west2`).
3. Open the **Rules** tab, paste in the contents of `firestore.rules`, and **Publish**. Edit the expiry date in that file first so it's a few days after your collection day.

### 2. Connect the app
1. **Project settings (gear icon) → General → Your apps → Web (`</>`)**. Register an app (skip "Firebase Hosting" if offered).
2. Copy the `firebaseConfig` values into the `CONFIG.firebase` block near the top of the script in `index.html`.
3. Optionally change `startCenter` (where the map opens) and `countryCodes` in the same block.

### 3. Put it online (pick one)

**Option A: Netlify Drop (easiest, no tools to install)**
Put `index.html` in a folder, go to https://app.netlify.com/drop and drag the folder on. You get an HTTPS link straight away. You can rename the site to something obscure in its settings.

**Option B: Firebase Hosting (keeps everything in one place)**
```
npm install -g firebase-tools
firebase login
firebase deploy --only hosting,firestore --project YOUR_PROJECT_ID
```
Your link will be `https://YOUR_PROJECT_ID.web.app`.

### 4. Test
Open the link on two phones, add an address on one, and watch it appear on the other. Tell people to "Add to Home Screen" if they like.

## On the day
- Collection team: **+ Address** for new requests, tap a pin then **Mark collected** when done, then **+ Pile** to record where the pile is (it uses GPS, drag the pin if it's off).
- Van team: tap a pile, then **Directions** to navigate, **Mark picked up** when loaded. The header shows trees still waiting in piles.
- Mistake? Tap the grey/green pin and choose **Undo**. Wrong entry? **Delete**.
- Signal drops: changes are queued on the phone and sync when it comes back (status shows "Offline – will sync").

## Afterwards (please do this: it's people's home addresses)
- Firebase console → Firestore → delete the `items` collection (or delete the whole project under Project settings).
- Delete the Netlify site / Hosting site.

## Notes
- The Firebase API key in the page is meant to be public. What protects the data is the obscure URL plus the expiring rules in `firestore.rules`.
- Address search uses OpenStreetMap's free Nominatim service (with Photon as a backup), which is fine for light use like this. Map tiles are OpenStreetMap's own once the page is hosted. If you open `index.html` straight from your computer, OpenStreetMap blocks its tiles, so request a free key from https://www.carto.com/basemaps/apikey/ (non-commercial use, no account) and paste it into `cartoKey` in the config block.
