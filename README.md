# WSO Practice Materials

Practice site for the Wayzata Symphony Orchestra. Static HTML, no build step, hosted on GitHub Pages at [wso.learndoteach.org](https://wso.learndoteach.org).

## What it does
- **Landing page** — Lists upcoming concert pieces grouped by concert
- **Song pages** — YouTube embed with loop controls and adjustable speed for practicing specific passages
- **Virtual concert** — Plays all pieces in program order on loop
- **Polyrhythm practice** — 3-against-4 metronome for Mahler 2 m. 250

## Quick start
```sh
# Clone
git clone <repo-url>
cd wso

# Develop — just open any HTML file in a browser
open index.html

# Or run a tiny local server (for fetch() to work)
python3 -m http.server 8000
# then visit http://localhost:8000
```

No build step, no dependencies, no package manager. Edit, refresh, done.

## How it deploys
Push to `main` → GitHub Pages rebuilds automatically → live at `wso.learndoteach.org` within ~30 seconds.

## Common tasks
- **Update a loop timestamp** → Edit `songs.json`, commit, push
- **Add a new song** → Add an entry to `songs.json` under `"songs"`, set `order` field
- **Add a new concert** → Add entry under `"concerts"` in `songs.json`, reference by key in songs

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## File map
| File | Purpose |
|---|---|
| `index.html` + `index.css` | Landing page |
| `song.html` + `song.css` | Song practice page (loop controls) |
| `concert.html` | Virtual concert (plays all pieces on loop) |
| `polyrhythm.html` | 3-against-4 polyrhythm practice tool |
| `songs.json` | All song data, concert metadata, loop times |
| `feather.svg` | Decorative feather icon used across pages |
| `CNAME` | GitHub Pages custom domain config |

## Documentation
- [ARCHITECTURE.md](ARCHITECTURE.md) — How the pieces fit together
- [CONTRIBUTING.md](CONTRIBUTING.md) — How to make changes safely
- [DATA.md](DATA.md) — `songs.json` schema reference
- [aesthetics.md](aesthetics.md) — Design language
- [looper.md](looper.md) — Song page (core practice tool) notes
- [virtualconcert.md](virtualconcert.md) — Concert page notes
- [polyrhythm.md](polyrhythm.md) — Polyrhythm tool notes
- [bugfixes.md](bugfixes.md) — History of fixes
- [unfixedbugs.md](unfixedbugs.md) — Known issues

## Pre-commit hook
A local hook validates `songs.json` before every commit. Set it up after cloning:
```sh
cat > .git/hooks/pre-commit <<'EOF'
#!/bin/sh
if git diff --cached --name-only | grep -q 'songs\.json'; then
  python3 -c "import json; json.load(open('songs.json'))" 2>/dev/null
  if [ $? -ne 0 ]; then
    echo "ERROR: songs.json is not valid JSON. Fix before committing."
    exit 1
  fi
fi
EOF
chmod +x .git/hooks/pre-commit
```
