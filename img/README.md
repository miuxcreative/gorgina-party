# /img

Drop Gorgina photos in here, then reference them by filename in the
CONFIG block at the bottom of gorgina-rsvp.html and gorgina-flyer.html:

    photos: {
      hero:  "img/your-file.jpg",
      strip: ["img/1.jpg", "img/2.jpg"],
      tiles: { bar: "img/bar.jpg", ... }
    }

Any size works — the page crops to fit. JPG or PNG, keep individual
files under ~2MB so the page loads fast on mobile data at the party.
