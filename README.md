# Gorgina

Gina's 32nd birthday — backyard day party, Sunday August 23, two to eight.

- `index.html` — the RSVP page (this is what GitHub Pages serves at the root)
- `flyer.html` — the shareable 1080×1350 flyer, with PNG export
- `img/` — drop party photos here, referenced from the `CONFIG` block in both HTML files
- `SETUP-google-sheet.md` — wiring the RSVP form to a Google Sheet

## Going live with GitHub Pages

After pushing this repo to GitHub:

1. On the repo page: **Settings → Pages**
2. Under **Build and deployment → Source**, choose **Deploy from a branch**
3. Branch: **main**, folder: **/ (root)** → **Save**
4. GitHub gives you a URL in a minute or two, usually:
   `https://YOUR-USERNAME.github.io/gorgina-party/`
5. The flyer lives at the same URL + `/flyer.html`

Every push to `main` updates the live site automatically — no rebuild step.

## Before you send invites

- [ ] Real address in `CONFIG.address` (both `index.html` and `flyer.html`)
- [ ] Google Sheet endpoint pasted into `CONFIG.endpoint` (see `SETUP-google-sheet.md`)
- [ ] Photos dropped into `img/` and referenced in `CONFIG.photos`
- [ ] Test an RSVP end to end and check it lands in the sheet
