# Song Looper (song.html) — Development Notes

## What it is
The core practice page. Each song has a YouTube embed with loop controls — musicians can define start/end times and practice specific sections on repeat at adjustable speed.

## Features
- YouTube iframe embed with loop sections loaded from `songs.json`
- Each loop has start/end time inputs and a "loop section" button
- Loops support both tuple format `["0:24", "2:13"]` and object format `{"start": "0:24", "end": "2:13", "label": "Andante, mm. 0–29"}`
- Labels display above each loop's controls when present
- "+" button adds a new loop starting where the last one ended
- Speed control (0.25x–2x)
- "play piece" button for full playback

## parseTime format support
Supports three formats:
- `SS` — raw seconds
- `M:SS` — minutes and seconds
- `H:MM:SS` — hours, minutes, seconds (added for long pieces)

## YouTube slow playback warble
- YouTube's `setPlaybackRate()` uses built-in time-stretching that causes warble on sustained tones (strings, winds)
- Can't access YouTube iframe's audio pipeline due to cross-origin restrictions
- `preservesPitch` property exists on HTML5 video/audio but inaccessible inside YouTube iframe
- Possible solution: host audio files directly and use Web Audio API with proper time-stretching (like SoundTouchJS)
- Alternative: link out to dedicated practice apps (Amazing Slow Downer, Anytune)
- The concept of `preservesPitch = false` + pitch-shift post-processing was discussed but is blocked by the same cross-origin wall

## songs.json structure
```json
{
  "concerts": {
    "transformations": { "name": "transformations", "date": "may 8, 2026" }
  },
  "songs": {
    "slug": {
      "title": "composer — piece name",
      "videoId": "YouTubeId",
      "concert": "transformations",
      "order": 1,
      "loops": [...]
    }
  }
}
```

Concert metadata lives in `songs.json` alongside song data — single file for all conductor updates. Originally concert dates were hardcoded in `index.html` JavaScript; moved to JSON for maintainability.

## Key architecture decisions
- `createLoopElement(startVal, endVal, label)` — single function for building loop DOM, used both for JSON-loaded loops and user-added "+" loops
- Labels are siblings before the `.loop` div (not children), for consistent flex layout
- Loop button has `min-width: 8em` so "stop" text doesn't shrink the row
- `#loops` and `.loop` both `width: 100%` for consistent alignment
- YouTube player state managed with `playerReady` flag — all controls silently ignore clicks until ready
- `onStateChange(ENDED)` resets `playActive` and button text
- `onError` disables controls (`playerReady = false`)
- `checkLoop` has try/catch for player errors, and guards against `end <= start`
- Loop interval only runs when `loopActive` is true (not during full playback)

## Pre-commit hook
A git pre-commit hook validates `songs.json` as valid JSON before every commit:
```sh
#!/bin/sh
if git diff --cached --name-only | grep -q 'songs\.json'; then
  python3 -c "import json; json.load(open('songs.json'))" 2>/dev/null
  if [ $? -ne 0 ]; then
    echo "ERROR: songs.json is not valid JSON. Fix before committing."
    exit 1
  fi
fi
```
Lives in `.git/hooks/pre-commit` (not tracked by git, local to each clone).
