# CLAUDE.md

Context for working on this project. Read this before making changes.

## What this is

A companion/scorepad web app for a tabletop game. It's a single self-contained
`index.html` — plain HTML, CSS, and vanilla JavaScript, no framework, no build
step. It was converted from an original React Native (Expo) app; the RN source
is not used anymore, the `.html` is the source of truth.

There are three tabs:
1. **Home** — a player-vs-type scoring grid plus a point tracker.
2. **Cards** — an ability-card browser with filtering by type icon and a
   fullscreen card view.
3. **Rulebook** — the game's rulebook PDF in an iframe.

## Core design constraints (do not violate without asking)

- **Private, single-device, no backend.** Each player runs the app on their own
  device and tracks only their own data. **Nothing syncs between players.** Do
  NOT add a server, accounts, login, room codes, realtime sync, or any backend.
  This is a deliberate product decision, not a missing feature.
- **No build step.** Keep it as a single `index.html` that runs by opening it
  (over a local server or a deployed URL). Don't introduce bundlers, npm
  dependencies, or a framework unless explicitly asked.
- **Vanilla JS only.** The app uses a small `el()` DOM helper and a single
  `state` object with a `render()` loop. Follow that existing pattern rather
  than introducing a library.

## How the code is structured (inside index.html)

- `ICONS` — the 8 type icons (key, label, image path).
- `ABILITY_IMAGES` — card data; each has `src`, `name`, and `tags` (array of
  icon keys it belongs to).
- `TAB_ICONS`, `RULEBOOK_PDF` — asset paths for the tab bar and PDF.
- `state` — single source of truth. Durable fields: `playerNames`, `tableData`
  (8x8, each cell cycles 0→1→2→3), `points`, `selectedIcons`. Transient fields
  (NOT persisted): `activeTab`, `fullscreenCard`, `modal`, `inputValue`.
- `render()` — rebuilds the active tab from `state`, then calls `saveState()`.
- Persistence: `saveState()` / `loadState()` use `localStorage` under the key
  `gameCompanion.v1`. Load is validated per-field so corrupt/old data can't
  crash startup. The player-name input saves on `oninput` directly (it
  intentionally does not re-render, to preserve keyboard focus).
- Dialogs: native `alert`/`confirm` were replaced with in-page `showNotice()`
  and `showConfirm()` for reliable mobile behavior.
- Images use an `imgOrFallback()` helper that swaps to a text fallback if the
  asset is missing, so the app stays usable even without art.

### Gotchas

- **Reset button z-index:** the reset button must stay above the `.content`
  scroll area (currently `z-index: 50`). The scroll container has
  `overflow: auto` and will swallow clicks in the top-right corner if the
  button's z-index drops.
- **Storage version key:** if you change the shape of any persisted field
  (e.g. add a 9th player), bump `STORAGE_KEY` to `gameCompanion.v2` so old
  saved blobs aren't misread.
- **file:// limits:** some browsers block localStorage and service workers on
  `file://`. Test over a local server or a deployed URL, not by double-clicking.

## Expected asset layout

The code references these relative paths. Real local filenames may differ
slightly (capitalization, extra words, .jpg vs .png) — reconcile them against
the actual files in `assets/` and fix mismatches in `index.html`.

```
assets/
  Scorpion_ICON.png  Poison_ICON.png  Tree_ICON.png  Wood_ICON.png
  Glass_ICON.png     Bull_ICON.png    Horned_ICON.png  Rain_ICON.png
  App_Background.mp4
  App_Rulebook.pdf
  Players/      (Scorpion.png, Poison.png, Tree.png, Wood.png,
                 Glass.png, Bull.png, Horned.png, Rain.png)
  Abilities/    (Ability_*.png — see ABILITY_IMAGES in index.html for the list)
  Buttons/      (Reset_Button.png, Home_Tab.png, Cards_Tab.png, Rulebook_Tab.png)
```

## Roadmap (rough order)

1. **Reconcile assets.** Compare paths in `index.html` to the real files in
   `assets/` and fix any mismatches. Confirm everything renders.
2. **Compress the background video.** It's likely large; compress hard or fall
   back to a static image on small screens so it isn't slow on mobile data.
3. **Deploy to a static host** (Netlify, Vercel, Cloudflare Pages, or GitHub
   Pages) to get a real HTTPS URL. HTTPS is required for the PWA step.
4. **Add the PWA layer** so players can install it and use it offline:
   - `manifest.json` (name, icons at 192 & 512, theme color, `display: standalone`)
   - a service worker with **versioned caching** (bump the cache version on each
     deploy or users keep getting stale files)
   - iOS-specific tags in `index.html`: `apple-touch-icon` and
     `apple-mobile-web-app-capable` (iOS doesn't reliably read the manifest icon)
   - a small "tap Share → Add to Home Screen" hint, since iOS shows no auto
     install prompt
   - Note: avoid caching the large video for offline (storage bloat). iOS may
     evict the cache after periods of disuse — re-cache critical assets on launch.

## Things to be careful about

- If the game's art, card text, and rulebook are someone else's intellectual
  property, confirm there's the right to host them publicly before deploying.
- Keep touch targets usable on phones — the grid cells are small (35px); enlarge
  if testing on real devices shows they're hard to tap.
- Test on real iOS Safari specifically; it has the most quirks.
