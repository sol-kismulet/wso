# Unfixed Bugs & Known Issues

## Active issues

### Cache-busting on static assets
CSS and JS files are linked without version parameters (`song.css`, not `song.css?v=2`). After a deploy, mobile browsers may serve stale cached files. This could explain times when the user saw old layouts despite pushes to main. Would require a build step or manual versioning to fix.

### Touch targets below Apple's 44px guideline
- `.loop .loop-btn`: ~29px tall
- General buttons: ~38px tall
- Time inputs: ~24px tall
On phones, tapping the right button in a row of small elements is fiddly. Should increase padding or min-height.

### Coleman loops are placeholder data
`["0:00", "1:00"], ["1:00", "2:00"], ...` are arbitrary 1-minute intervals, not real musical sections. Waiting for conductor to provide actual timestamps and labels.

### Mahler #27 end time is a placeholder
Last loop ends at `40:00` — this is an estimate. Conductor hasn't confirmed the actual end of the cello part in the new video.

### Virtual concert plays full YouTube videos
The Mahler video contains all movements but we only use the 1st movement (starts at ~17:12). The concert page plays the entire video from the start. Could add start/end offsets per song if needed.

### Polyrhythm page — iOS playback unconfirmed
The Web Worker + pre-rendered buffer architecture (v4) hasn't been tested on the user's iPhone/iPad. Previous versions were choppy. Needs real-device testing.

## Low priority / edge cases

### parseTime doesn't validate input
User-typed values in loop time inputs aren't validated. `"abc"` parses as 0, empty string parses as 0. Not a crash, but confusing if someone accidentally clears a field.

### YouTube API failure is silent (song.html)
If the YouTube iframe API script fails to load (ad blocker, firewall), the page renders normally with controls but nothing works. No error message shown.

### No favicon
Browsers 404 on `/favicon.ico` every page load. Minor but shows in server logs.

### song.html still has inline `<style>` block
`#video-wrapper`, `.back`, and `.loop-label` styles are inline in `song.html` rather than in `song.css`. Intentional (page-specific styles) but inconsistent with the `index.css` extraction.
