# Wiring the RSVP form to a Google Sheet

About 5 minutes, one time. No plugins, no third-party service, no monthly fee.

---

## 1. Make the sheet

1. Go to [sheets.new](https://sheets.new) and name it **Gorgina RSVPs**.
2. Leave row 1 empty — the script writes the headers itself on the first submission.

## 2. Add the script

In the sheet: **Extensions → Apps Script**. Delete whatever's in the editor and paste this in:

```javascript
// Gorgina RSVP → Google Sheet
// Receives form posts, appends a row, and saves any uploaded photo to Drive.

const SHEET_NAME = 'RSVPs';
const PHOTO_FOLDER_NAME = 'Gorgina RSVP Photos';

const FIELDS = [
  'timestamp', 'name', 'attending', 'party_size', 'kids',
  'arrival', 'transport', 'dietary', 'song', 'contact', 'note', 'photo'
];

const HEADERS = [
  'Submitted', 'Name', 'Attending', 'Party size', 'Kids',
  'Arriving', 'Getting here', 'Dietary', 'Song request', 'Contact', 'Note for Gina', 'Photo'
];

function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.waitLock(20000);

  try {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    let sheet = ss.getSheetByName(SHEET_NAME);

    if (!sheet) {
      sheet = ss.insertSheet(SHEET_NAME);
    }

    if (sheet.getLastRow() === 0) {
      sheet.appendRow(HEADERS);
      sheet.getRange(1, 1, 1, HEADERS.length)
           .setFontWeight('bold')
           .setBackground('#FFD23F');
      sheet.setFrozenRows(1);
    }

    const params = (e && e.parameter) ? e.parameter : {};
    const photoUrl = savePhoto_(params.photo, params.name);

    const row = FIELDS.map(function (key) {
      if (key === 'timestamp') {
        return new Date();
      }
      if (key === 'photo') {
        return photoUrl;
      }
      return params[key] || '';
    });

    sheet.appendRow(row);

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', message: err.message }))
      .setMimeType(ContentService.MimeType.JSON);

  } finally {
    lock.releaseLock();
  }
}

// Decodes the data: URL the page sends and saves it into a Drive folder.
// Returns a viewable link, or '' if no photo was submitted.
function savePhoto_(dataUrl, name) {
  if (!dataUrl) return '';

  const match = String(dataUrl).match(/^data:(image\/[a-zA-Z0-9.+-]+);base64,(.+)$/);
  if (!match) return '';

  try {
    const mimeType = match[1];
    const ext = mimeType.split('/')[1].replace('jpeg', 'jpg');
    const bytes = Utilities.base64Decode(match[2]);
    const safeName = (name || 'guest').toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/(^-|-$)/g, '');
    const blob = Utilities.newBlob(bytes, mimeType, safeName + '-' + Date.now() + '.' + ext);

    const existing = DriveApp.getFoldersByName(PHOTO_FOLDER_NAME);
    const folder = existing.hasNext() ? existing.next() : DriveApp.createFolder(PHOTO_FOLDER_NAME);

    const file = folder.createFile(blob);
    file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
    return 'https://drive.google.com/uc?id=' + file.getId();
  } catch (err) {
    return 'upload failed: ' + err.message;
  }
}

function doGet() {
  return ContentService.createTextOutput('Gorgina RSVP endpoint is alive.');
}
```

Save it (⌘S). Name the project anything.

> **Already have this deployed without photo support?** Replace the script with
> the version above and redeploy (see the version-bump note in step 3) — the
> URL stays the same, nothing else changes. The next deploy will ask you to
> re-authorize because the script now touches Drive, not just Sheets; that's
> expected, click through it the same way as the first authorization.

## 3. Deploy it

1. **Deploy → New deployment**
2. Click the gear next to "Select type" → **Web app**
3. Set:
   - **Description:** anything
   - **Execute as:** *Me*
   - **Who has access:** ***Anyone*** ← this one matters. Not "Anyone with Google account."
4. **Deploy** → authorize when Google asks.
   Google will warn you the app isn't verified. Click **Advanced → Go to [project name] (unsafe)**. It's your own script; this is normal.
5. Copy the **Web app URL**. It looks like:
   `https://script.google.com/macros/s/AKfyc.../exec`

## 4. Paste it into the page

Open `gorgina-rsvp.html`, find the CONFIG block near the bottom, and drop the URL in:

```javascript
const CONFIG = {
  endpoint: "https://script.google.com/macros/s/AKfyc.../exec",
  ...
};
```

Submit a test RSVP. A row should appear in the sheet within a second or two.
If you attach a photo, the sheet's **Photo** column gets a Drive link instead
of the image itself — the actual files land in a Drive folder called
**Gorgina RSVP Photos** (created automatically on the first upload).

> **If you edit the script later,** you have to **Deploy → Manage deployments → edit (pencil) → Version: New version → Deploy.** Saving alone doesn't update the live URL. This trips up everyone once.

---

## Where to host the page

The file is fully self-contained — one HTML file, no build step. Any of these work:

| Option | How | Cost |
|---|---|---|
| **Netlify Drop** | Drag the file onto [app.netlify.com/drop](https://app.netlify.com/drop) | Free, instant, gives you a URL |
| **GitHub Pages** | Push to a repo, enable Pages | Free |
| **Vercel** | `vercel deploy` in the folder | Free |
| **Your own domain** | Drop it at something like `gorgina.madebymiu.com` | You already own the domain |

Rename the file to `index.html` before uploading so the URL stays clean.

---

## Config reference

Everything editable lives in one block at the bottom of `gorgina-rsvp.html`:

| Key | What it does |
|---|---|
| `endpoint` | The Apps Script URL above. Blank = form runs in demo mode and logs to console. |
| `address` | Full address. Shows in the details, the copy button, and the calendar file. |
| `hostPhone` | Optional. If filled, adds a "text us" line in the footer. |
| `rsvpBy` | The date shown above the form. |
| `dressCode` | The "Wear" line. |
| `bringLine` | The "Bring" line. |
| `parkingSpots` | Number quoted in the parking paragraph. |
| `kidsWelcome` | `false` hides the kids dropdown and swaps the copy to a grown-ups-only note. |
| `gifts` | `"none"` or `"welcome"`. |
| `startISO` / `endISO` | Used by the countdown and the calendar file. Already set to Aug 23, 12–8 ET. |

---

## Flyer notes

`gorgina-flyer.html` opens in a browser and renders at 1080 × 1350 (Instagram 4:5, also fine for texting and printing).

1. Paste your live RSVP URL in the field at the top → **Update QR**.
2. **Download PNG** for social, or **Print / Save PDF** for a physical copy.
3. To bake the link in permanently so you never retype it, set `RSVP_URL` at the top of the script block.

The QR and PNG export both pull small libraries from a CDN, so keep the browser online when exporting. If you're offline, screenshot works fine too.
