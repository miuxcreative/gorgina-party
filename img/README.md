# /img

Drop Gorgina photos in here, then reference them by filename in the
CONFIG block at the bottom of index.html and flyer.html:

    photos: {
      hero:  "img/your-file.jpg",       // used only if heroCollage is empty
      heroCollage: ["img/1.jpg", "img/2.jpg", ...],  // photo wall behind the hero, 6–10 looks best
      strip: ["img/1.jpg", "img/2.jpg"],
      tiles: { bar: "img/bar.jpg", ... }
    }

Any size works — the page crops to fit. JPG or PNG, keep individual
files under ~2MB so the page loads fast on mobile data at the party.
The same photo can be reused across heroCollage/strip/tiles — no need
to shoot each slot separately.
