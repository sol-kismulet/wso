# Bug Fixes — Complete History

## Round 1: Initial audit and cleanup

### Bugs fixed
1. **No early return when slug is missing (song.html)** — When no `?s=` parameter, the code showed "no song specified" but kept running. Clicking play/speed/+ buttons crashed because `player` was undefined. Fixed with `if/else` wrapping all logic in `init(slug)`.

2. **"+" button crashes on empty loops array (song.html)** — `loops[loops.length - 1]` was undefined if no loops existed. Added `if (loops.length === 0) return` guard.

3. **Speed/play controls crash before YouTube loads (song.html)** — `player.seekTo()` and `player.setPlaybackRate()` threw before YouTube API loaded. Added `playerReady` flag, all controls silently return if false.

4. **setInterval loop polling at 50ms (song.html)** — Ran 20x/second checking `getCurrentTime()`. Wasteful and could overshoot in background tabs. Changed to state-driven: interval only created in `onStateChange(PLAYING)` and cleared on any other state.

### Dead code removed
5. **feather.js** — Never included by any page. Defined `renderFeather()` that nothing called. Had a dangerously broad `svg path` CSS rule. Deleted.

6. **audiolooper.js** — Designed for `<audio>` elements, but site uses YouTube embeds. Never included. Deleted.

7. **Unused CSS classes (.footer-note, .lyrics) in song.css** — Didn't match any HTML element. Removed.

8. **Redirect HTML files** — `coleman-umoja.html`, `brahms-haydn-variations.html`, `mahler-symphony-2.html` were manual redirect files. Added maintenance overhead per song. Deleted.

### Structural improvements
9. **Concert metadata hardcoded in index.html** — `concertMeta` with dates was separate from `songs.json`. Moved into `songs.json` under a `"concerts"` key. Now a conductor email = one file edit.

10. **Duplicated loop-creation code (song.html)** — Same DOM-building logic existed in the JSON loader and the "+" button handler. Extracted into `createLoopElement(startVal, endVal, label)`.

11. **Missing Google Fonts link in song.html** — `song.css` referenced Cormorant Garamond but `song.html` never loaded the font. Direct navigation to song page got the wrong font. Added the `<link>` tag.

12. **Index.html CSS inline (~240 lines)** — Extracted to `index.css` for consistency with `song.css`.

## Round 2: Second audit

13. **Duplicate .loop-label styles** — Defined in both `song.css` and `song.html` inline `<style>` with conflicting color values (0.5 vs 0.45 opacity). Removed from `song.css`, kept in inline with `width: 100%` added.

14. **Interval leak in onStateChange** — If YouTube fired `PLAYING` twice without intervening state, a new `setInterval` was created without clearing the old one. Fixed: always `clearInterval(loopTimer)` before conditionally creating a new one.

15. **No error handling on index.html fetch** — If `songs.json` failed to load, the landing page silently showed nothing. Added `.catch()` handler showing "failed to load songs".

16. **Dead .glyph.feather-glyph selector (song.css)** — `.title .glyph.feather-glyph` didn't match any element. Removed.

17. **Stale comment (song.html)** — "Use timeupdate polling via onStateChange instead of setInterval" was inaccurate. Removed.

18. **Inline style on speed label** — `style="margin-right:0.5rem;"` moved to CSS class `.speed-label`.

## Round 3: Third audit

19. **playActive not reset when video ends (song.html)** — If user clicked "play piece" and video played to the end, button stayed as "stop" and `playActive` stayed true. Added `onStateChange(ENDED)` handler to reset both.

20. **Speed display inconsistency** — Page loaded showing "1x" but slider formatted as "1.00x". Changed initial HTML to "1.00x".

21. **End < start infinite seek stutter (song.html)** — If user edited end time to be before start time, `checkLoop` fired every 200ms seeking back endlessly. Added `if (end <= start) return` guard.

22. **Missing loops array crashes page (song.html)** — A song without `"loops"` in JSON threw `TypeError` on `song.loops.forEach()`. Changed to `(song.loops || []).forEach()`.

23. **YouTube onError not handled (song.html)** — If video was removed/private/embedding disabled, controls remained active on a broken player. Added `onError` handler: sets `playerReady = false`, stops loop, resets button.

24. **checkLoop has no error protection (song.html)** — If YouTube player entered error state during loop, `getCurrentTime()` could throw uncaught errors every 200ms. Added try/catch.

25. **checkLoop runs during full playback (song.html)** — `onStateChange(PLAYING)` created interval even when `loopActive` was false (during "play piece"). Added `&& loopActive` guard.

26. **No response.ok check on fetches** — `fetch` resolves on 404/500. Both pages' `.then(r => r.json())` would try to parse error HTML as JSON. Added `if (!r.ok) throw new Error(r.status)` to both `song.html` and `index.html`.

27. **Feather alt text (song.html)** — Was `alt="feather"` (announced by screen readers). Changed to `alt=""` (decorative) to match index page.

28. **Extra blank line in song.css** — Double blank line before `a` selector. Removed.

## Round 4: Trailing comma incident

29. **Trailing comma in songs.json** — After editing Brahms loops, a trailing comma was left on the last array element. Invalid JSON broke the entire site ("failed to load"). Fixed immediately. **Led to adding a git pre-commit hook** that validates `songs.json` before every commit.

## Round 5: Layout and data fixes from other sessions

30. **Loop row layout for mobile (song.css)** — Added flex-wrap, `width: 4.5em` on time inputs, `width: min(560px, 90vw)` on controls container. (Note: user reported this didn't take effect on iPhone because changes were on a branch, not main.)

31. **Loop label moved outside .loop div (song.html)** — Other session moved label from inside the flex container to a sibling before it, for consistent loop width. Added `#loops { width: 100% }`, `.loop { width: 100% }`.

32. **Loop button min-width (song.css)** — `min-width: 8em` on `.loop-btn` prevents layout shift when text changes between "loop section" and "stop".
