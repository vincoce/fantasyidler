# Lanternfall

A slow, no-click fantasy idle game. Your party walks the world map on its own, fights groups
of monsters, levels up and buys gear. You only make choices if you want to: talents, travel,
manual upgrades. A guild of up to 10 friends shares a weekly raid boss and daily blessings.

## Files

- `index.html` – the whole game
- `firebase-config.js` – your Firebase web config (online play)
- `firestore.rules` – security rules for Firestore

Without a filled-in `firebase-config.js`, the game still runs, but the realm stays in that browser.

## Setup (using the same Firebase project as Dripstone)

1. Make a new GitHub repository (for example `lanternfall`), upload `index.html`, and enable
   GitHub Pages for it.
2. Copy `firebase-config.js` from your Dripstone repository into this repository.
3. In Firebase → Firestore → Rules, paste the block between `Lanternfall start` and
   `Lanternfall end` into your existing rules, next to Dripstone's rules. Don't delete
   Dripstone's rules. Click **Publish**.
4. Authorized domains: if Lanternfall is on the same `yourname.github.io` domain as Dripstone,
   nothing to do. Otherwise add the domain in Authentication → Settings → Authorized domains.

## How saving works (and why progress can't be overwritten anymore)

Dripstone saved the cave as numbers (calcite, levels). Two devices with different numbers
fought over the same save, and the one that wrote last won.

Lanternfall never saves the numbers. A realm is:

- a seed and a start time, and
- the list of choices you made, each with a timestamp ("upgrade Torvin's weapon at 14:02").

Everything else is recalculated from those. Time passes the same on every device, and the
fights are fully deterministic, so the same choices always give the same realm, to the last
gold coin. Devices only ever *add* choices to the list (Firestore `arrayUnion`), so nothing can
be overwritten. If a phone was offline and made a choice an hour ago, the other devices rewind
to a snapshot from before that hour and replay, and everyone ends up in the same place.

Snapshots ("checkpoints") are stored too, so a device doesn't need to replay months of fights.
They're only used when they match the list of choices exactly, so a stale one can't do harm.

If you sign in on a browser that already has a *different* realm, the game asks which one to keep.
