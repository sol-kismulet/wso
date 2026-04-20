# Polyrhythm Practice Tool — Development Notes

## What it is
A 3-against-4 polyrhythm practice tool for Mahler Symphony No. 2, measure 250. Helps orchestra musicians internalize the feel of triplets against quarter notes.

## Current state
Under construction. Core functionality works but has had choppy playback issues on iOS Safari that we've been iterating on.

## Architecture (current, post-Gemini review)

### Audio engine
- **Pre-rendered AudioBuffers**: Each voice (click, high, bell, low, snap, drum) is generated once at init into an AudioBuffer using `audioCtx.sampleRate`. Beats play lightweight `AudioBufferSourceNode` copies (fire-and-forget, no pooling needed).
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

Defaults: layer 4 = low, layer 3 = high. Each layer independently selectable with preview-on-tap and per-layer mute.

## Iteration history (choppy playback on iOS)
1. **v1**: `setInterval(25ms)` + 100ms look-ahead + `OscillatorNode` per beat → choppy, throttled on mobile
2. **v2**: `setTimeout` recursion + 300ms look-ahead + `OscillatorNode` per beat → still choppy
3. **v3**: `setTimeout` + 500ms look-ahead + pre-rendered `AudioBuffers` → less choppy but 500ms input lag on tempo/mute changes
4. **v4 (current)**: Web Worker ticker + 120ms look-ahead + pre-rendered buffers + gain normalization → needs testing on mobile

## Gemini's review (key recommendations, all implemented)
- Web Worker ticker instead of main-thread setTimeout
- Reduced look-ahead from 500ms to ~120ms for responsive controls
- Removed DynamicsCompressor, use 0.5 gain normalization per layer (less DSP overhead)
- Time-stamped visual event queue instead of polling
- AudioContext.onstatechange for iOS suspend/interrupt
- Measure duration L with L/4 and L/3 intervals to prevent float drift
- Fire-and-forget AudioBufferSourceNodes (don't pool — creation cost is negligible)
- Use audioCtx.sampleRate for buffer generation

## Gemini's additional notes
- iOS "silent switch" bypassed by creating AudioContext inside user gesture
- iOS sample rate can fluctuate (44.1kHz vs 48kHz) depending on other apps — always use `audioCtx.sampleRate` at generation time
- DynamicsCompressorNode is computationally expensive on mobile Safari — avoid unless truly needed

## Still needs testing
- User hasn't confirmed v4 (Web Worker) works on mobile iOS
- Beat 1 simultaneous playback (both layers) — should be clean with 0.5 gain normalization but unconfirmed on device

## File structure
- `polyrhythm.html` — self-contained page, uses `song.css` for base styles
- Index page has a "polyrhythm practice" link at the bottom (in `<nav class="tools">`, styled in `index.css`)
