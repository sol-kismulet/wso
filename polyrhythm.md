# Polyrhythm Practice Tool — Development Notes

## What it is
A 3-against-4 polyrhythm practice tool for Mahler Symphony No. 2, measure 250. Helps orchestra musicians internalize the feel of triplets against quarter notes.

## Current state
Under construction. Core functionality works but has had choppy playback issues on iOS Safari that we've been iterating on.

## Architecture (current, post-Gemini review)

### Audio engine
- **Pre-rendered AudioBuffers**: Each voice (click, high, bell, low, snap, drum) is generated once at init into an AudioBuffer. Beats play lightweight `AudioBufferSourceNode` copies (fire-and-forget, no pooling needed).
- **Web Worker ticker**: Inline Blob worker sends `postMessage('tick')` every 25ms on a separate thread, immune to main-thread congestion and mobile Safari throttling.
- **Look-ahead**: 120ms. Scheduler runs on each worker tick, scheduling any beats falling within that window using precise `AudioContext.currentTime`.
- **Audio routing**: Each layer routes through its own `GainNode` (0.5 gain) directly to destination. Two layers sum to max 1.0 — no clipping, no compressor needed.
- **Timing math**: Measure duration `L = (60/bpm) * 4`. Layer 4 interval = `L/4`, layer 3 interval = `L/3`. Derived from constant L to prevent cumulative float drift.

### Visual sync
- `requestAnimationFrame` consumes a time-stamped beat queue `[{time, layer, index}]`. Scheduler pushes events, rAF consumes them when `AudioContext.currentTime` reaches each timestamp.

### iOS handling
- `AudioContext.onstatechange` monitors for `interrupted`/`suspended` states (app switch, phone call) and auto-stops.
- AudioContext created/resumed inside user gesture (button click) for iOS autoplay policy.

## Six voices (per-layer selectable)
| Voice | Waveform | Frequency | Character |
|---|---|---|---|
| click | square | 800Hz | sharp percussive |
| high | square | 1400Hz | bright snap |
| bell | sine | 523Hz (C5) | warm sustain |
| low | sine | 262Hz (C4) | deep tone |
| snap | triangle | 2000Hz | crisp tick |
| drum | square | 200Hz | punchy thump |

Defaults: layer 4 = low, layer 3 = high.

## Known issues / still being worked on
1. **Choppy playback on iOS** — Has been the main battle. Went through multiple iterations:
   - v1: `setInterval(25ms)` + 100ms look-ahead + OscillatorNode per beat → choppy, throttled
   - v2: `setTimeout` recursion + 300ms look-ahead + OscillatorNode per beat → still choppy
   - v3: `setTimeout` + 500ms look-ahead + pre-rendered AudioBuffers → less choppy but input lag
   - v4 (current): Web Worker ticker + 120ms look-ahead + pre-rendered buffers → needs testing
2. **Beat 1 masking** — When both layers fire simultaneously. Solved by removing DynamicsCompressor (was expensive on mobile) and using 0.5 gain per layer instead.
3. **User hasn't confirmed current version works on mobile** — Still needs testing after the Web Worker rewrite.

## Gemini's review (key recommendations implemented)
- ✅ Web Worker ticker instead of main-thread setTimeout
- ✅ Reduced look-ahead from 500ms to ~120ms for responsive controls
- ✅ Removed DynamicsCompressor, use gain normalization
- ✅ Time-stamped visual event queue instead of polling
- ✅ AudioContext.onstatechange for iOS suspend
- ✅ Measure duration L with L/4 and L/3 intervals
- ✅ Fire-and-forget AudioBufferSourceNodes (don't pool)
- ✅ Use audioCtx.sampleRate for buffer generation (already was)

## Features
- Per-layer sound selection (6 voices, independently assignable)
- Preview on tap (hear the sound before starting)
- Per-layer mute
- Adjustable tempo (40–200 BPM)
- Visual beat indicators (blue circles for 4, warm circles for 3)
- Back link to index page
- Under construction notice at top

## File structure
- `polyrhythm.html` — self-contained page, uses `song.css` for base styles
- Index page has a "polyrhythm practice" link at the bottom (in a `<nav class="tools">` section, styled in `index.css`)

## Discussion notes from this session

### Warble on YouTube slow playback
- YouTube's `setPlaybackRate()` uses built-in time-stretching that causes warble on sustained tones
- Can't access YouTube iframe's audio pipeline due to cross-origin restrictions
- `preservesPitch` property exists on HTML5 video/audio but inaccessible inside YouTube iframe
- Best option for clean slow practice: host audio files directly and use Web Audio API with proper time-stretching (like SoundTouchJS)
- Alternative: link out to dedicated practice apps (Amazing Slow Downer, Anytune)

### Cache-busting
- Static assets (CSS, JS) have no version parameters
- GitHub Pages handles cache headers but mobile browsers cache aggressively
- Not yet addressed — would require a build step or manual versioning

### parseTime supports H:MM:SS
- Added support for hour-format timestamps (`1:02:30`) in addition to `M:SS` and raw seconds
- 3 colons = hours, 2 = minutes, 1 = raw seconds
