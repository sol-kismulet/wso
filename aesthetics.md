# Aesthetics & Design — Development Notes

## Overall design language
- Dark theme: black background (#000), white text (#fff)
- Font: Cormorant Garamond (Google Fonts) — serif, elegant, lowercase
- Minimal, understated — let the content breathe
- Consistent across all pages (index, song, polyrhythm, concert)

## Index page (index.html + index.css)
- Header: "wayzata symphony orchestra" in large lowercase with letter-spacing
- Subtitle: "practice pages" in italic, faded
- Feather SVG icon with falling animation (3s, drops from 20rem above with bounce)
- Golden glow easter egg: after 20 seconds, feather gets a golden shimmer that fades to warm red (`goldenGlow` keyframe)
- Concert sections: name + date, songs listed with composer name dimmed
- Song links: left border highlight on hover, subtle underline separators
- Footer: "wso" in very faded text
- Tools nav: "polyrhythm practice" and "virtual concert" links below song list, styled subtly to not compete with main content
- `fadeUp` animation on load for staggered entrance

## Song page (song.html + song.css)
- Title: song name in 3rem with feather icon inline
- "← back" link fixed top-left
- Video wrapper: `min(560px, 90vw)` width, 16:9 aspect ratio
- Loop rows: flex-wrap, centered, with `width: 100%` for consistent alignment
- Loop labels: italic, faded (0.45 opacity), 0.08em letter-spacing, positioned as siblings above their loop controls
- Loop button: `min-width: 8em` so "stop" doesn't shrink the row
- Time inputs: `4.5em` width, centered text
- Speed display: "1.00x" format
- Base font size: 2rem on body (large, readable)
- Desktop (900px+): vertically centered layout

## Inline styles still in song.html
- `#video-wrapper`, `.back`, `.loop-label` are in an inline `<style>` block in `song.html`
- These are page-specific styles that don't belong in the shared `song.css`

## CSS organization
- `index.css` — index page only (extracted from inline)
- `song.css` — shared base styles used by song.html, polyrhythm.html, concert.html
- Page-specific styles live in inline `<style>` blocks

## Feather icon
- `feather.svg` — hand-traced SVG (24.5KB)
- Used as `<img>` tag on all pages (not inline SVG)
- Decorative: `alt=""` on all pages
- Index page: animated drop + golden glow easter egg
- Song/polyrhythm/concert pages: static, inline with title

## Mobile considerations
- All width constraints use `min()` for responsive sizing
- Loop rows flex-wrap so button drops below inputs on narrow screens
- Touch targets: loop buttons are small (~29px tall, below Apple's 44px guideline) — noted but not yet addressed
- `position: fixed` on back link — works but can behave oddly on iOS when keyboard is open

## Color palette
- Background: #000
- Text: #fff
- Dimmed text: rgba(255,255,255,0.35–0.5)
- Links (song page): #9cd8ff
- Song link hover: white with left border rgba(255,255,255,0.5)
- Loop labels: rgba(255,255,255,0.45)
- Polyrhythm layer 4 (blue): #8cb4d8
- Polyrhythm layer 3 (warm): #d8a08c
- Buttons: #ddd text, #666 border on dark background
- Footer: rgba(255,255,255,0.15)

## Typography
- Cormorant Garamond: weights 300, 400, 500, 600 + italic 300, 400
- Lowercase throughout (text-transform: lowercase on headers)
- Letter-spacing: 0.02–0.15em depending on context
- Title sizes: 2.4rem (index h1), 2.8rem (desktop), 3rem (song title)
