# Game Companion

A private, single-device companion app for a tabletop game: a player/type
scoring grid, a point tracker, an ability-card browser, and the rulebook.

Built as a single `index.html` — plain HTML/CSS/vanilla JS, no framework, no
build step. Each player runs it on their own device; nothing syncs between
players. Game data is saved locally in the browser (`localStorage`).

## Run it

Because browsers restrict `localStorage` and service workers on `file://`,
serve it over a local web server rather than double-clicking the file:

```bash
# from the project folder, pick whichever you have:
python3 -m http.server 8000
# then open http://localhost:8000
```

```bash
npx serve .
```

## Project layout

```
index.html      the entire app
assets/         icons, player & ability art, buttons, background video, rulebook PDF
CLAUDE.md       context & roadmap (read this before making changes)
README.md       this file
```

See `CLAUDE.md` for the expected `assets/` structure, design constraints, and
the deployment / PWA roadmap.

## Deploy

It's a static site, so any static host works (Netlify, Vercel, Cloudflare
Pages, GitHub Pages) and provides free automatic HTTPS. HTTPS is required if
you later add the PWA (installable / offline) layer.
