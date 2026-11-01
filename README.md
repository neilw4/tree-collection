# Christmas tree collection map

One-page web app: add addresses for tree collection, mark them collected, drop piles, mark piles picked up. Everything shows live on a shared map.

- **Green tree** = a house with a tree to collect; **stack of trees** = a pile waiting for the van. Both turn grey once collected or picked up.
- The number on each icon is the number of trees (round badge: lighter green for houses, darker green for piles; greyed out once done)
- Tap an icon to mark it collected/picked up (and undo), **Edit** it (address, note, tree count, move the pin), get directions, or delete it
- Houses and piles can both carry a note, e.g. "tree is at the side gate" or "outside no. 12"
- The **Show** checkboxes at the top hide/show addresses, piles, and collected items (to declutter the map)
- The base map is deliberately greyscale so the tree icons stand out (one CSS line in `index.html` if you'd like colour back)

## Activity log
- **📜 Log** (top bar) lists what happened, newest first: adds, edits, collected / picked up, undo, deletes and spreadsheet imports, with the time and the name typed in the box at the top of the panel (stored on that device only).
- Each entry has an **Undo** button: it reverses a collected/picked-up mark or undoes an add, and a delete has **Restore** to bring the entry back. Edits are logged but can't be undone.
- Estates with no entries are hidden from the Estates list and the estate suggestions, and are removed automatically after you next add, edit or delete something.

## Estates
- When adding or editing a house **or a pile** you can give it an **estate** and a **pickup time**. Pick an existing estate from the list or type a new name and it is created.
- The app suggests the estate for you: if the address you typed contains an existing estate's name, or the pin lands within about 60 m of a house in that estate, the estate field is filled in (and you can change it).
- On the map each estate is a pastel-coloured outline (every estate gets its own colour) around its houses with a label: name, pickup time and how many houses are collected (e.g. "Beech Park · 10:00am · 2/8"). It turns grey with a tick when every house is done. Tap the outline to edit the estate's name or pickup time, or to delete the estate (its houses stay).
- The **Outlines** checkbox in the Show row hides all the outlines. The **Estates ▾** button opens a list where each estate can be switched on or off separately (its outline, houses and piles); All/None buttons switch them all.

## Adding many houses from a spreadsheet
Tap **+ Address**, then **⬆ Add many from a spreadsheet** under the search box.
1. Upload an Excel file (.xlsx/.xls), a CSV or a TSV (tab-separated) file, **or** copy the cells in Excel or Google Sheets and paste them into the box, **or** paste a Google Sheets link (the sheet must be shared as "Anyone with the link can view", or published to the web; private sheets can't be read by a web page without a Google sign-in setup, so for those copy and paste instead).
2. The app guesses which column is which from the titles and the contents: a single address column, or separate house number, street, estate, town, county and Eircode columns, plus number of trees, notes, pickup time and latitude/longitude. Title rows above the header are skipped. Names, phone numbers and email columns are ignored. Check the "Holds" menus and fix anything wrong.
3. Optionally enter a town/county (added to every search so the right place is found) and a default estate for rows without one.
4. Tap Import. Each address is located one at a time (about 1-2 seconds each on the free search, so allow a few minutes for a big list).
   - Houses found only at street or estate level are placed approximately, fanned out around that spot, and show a dashed number badge. Tap one and choose Edit to drag it onto the house.
   - Addresses that can't be found are listed with a **Place on map** button; you can come back to the list from the upload link in the + Address sheet.
   - Rows that have latitude/longitude columns filled in are placed exactly; every other row is looked up on the map automatically.
   - Rows already on the map (same address text) are skipped, so re-uploading is safe.
   - The `sample-data/` folder has example sheets that parse and ones that don't; `sample-data/REPORT.md` shows what the app makes of each.

## Try it first (no setup)

Open `index.html` in a browser. With no Firebase config it runs in **demo mode** (data saved only in that browser). If nothing is stored yet it starts with sample addresses, piles and estates around Maynooth; tap **Clear sample data** in the banner to remove them. Note that GPS only works once the page is served over HTTPS, i.e. after you deploy. Map tiles from a local file need a free CARTO key (see Notes at the bottom).

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

## Optional: better address search with Google
The default address search (OpenStreetMap-based) often finds only the street for Irish addresses and can't search Eircodes. If you add a Google key, the page uses Google's geocoder first and falls back to the free search if Google fails.

1. In the Google Cloud console, create a project, attach a billing account, and enable **Maps JavaScript API** and **Geocoding API**. (Geocoding currently has a monthly free allowance; check Google's pricing page, and set a budget alert or quota cap.)
2. Create an API key. Restrict it to **Websites (HTTP referrers)** with your site's address, e.g. `https://neilw4.github.io/*`, and under API restrictions allow only those two APIs.
3. Paste the key into `googleKey` in the config block of `index.html`.

Notes: a page opened from a local file sends no referrer, so a referrer-restricted key only works on the hosted page. Check Google's terms before relying on this, as they limit using Google geocoding results on non-Google maps.

## Notes
- The Firebase API key in the page is meant to be public. What protects the data is the obscure URL plus the expiring rules in `firestore.rules`.
- Addresses from a spreadsheet are sent to the address search service (OpenStreetMap's Nominatim/Photon, or Google if you add a key) to be located. Address search uses OpenStreetMap's free Nominatim service (with Photon as a backup), which is fine for light use like this. Map tiles are OpenStreetMap's own once the page is hosted. If you open `index.html` straight from your computer, OpenStreetMap blocks its tiles, so request a free key from https://www.carto.com/basemaps/apikey/ (non-commercial use, no account) and paste it into `cartoKey` in the config block.
