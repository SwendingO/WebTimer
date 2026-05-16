# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WebTimer is a full-screen countdown timer for TV display, designed to help children stay on task. It's a **single self-contained file** (`index.html`) with all HTML, CSS, and JavaScript embedded — no build step, no dependencies, no package manager.

**Live site:** https://swendingo.github.io/WebTimer  
**Deployment:** `git push` → GitHub Pages auto-deploys in ~60 seconds

## Development

No build tools. Edit `index.html` directly and open it in a browser to test. Push to `main` to deploy.

## Architecture

Everything lives in `index.html`. Two screens toggled via the `hidden` class:

1. **Setup screen** — Duration picker with presets (5 min–2 hr) and custom ±30-second stepper. Last duration persists via `localStorage`.
2. **Timer screen** — Countdown with two display modes (`view: 'ring'` | `'block'`), switchable at runtime.

### State Object

```js
const state = {
  totalSeconds,         // set by user
  remainingSeconds,     // current countdown
  running,              // boolean
  view,                 // 'ring' | 'block'
  tickInterval,         // setInterval ID
  milestoneTriggered,   // 1-min flash fired?
  milestone15Triggered, // 15-sec flash fired?
  oscillator,           // Web Audio oscillator (siren)
  audioCtx,             // Web Audio context (siren)
  heartbeatCtx,         // Web Audio context (heartbeat)
  colorBurstTicks,      // counter for urgent heartbeat pulses
  lastColor,            // tracks color transitions
}
```

### Color Progression

Colors are **proportional to total duration** (not fixed times):

| % remaining | Color  |
|-------------|--------|
| > 75%       | Green `#4ade80` |
| 50–75%      | Yellow `#facc15` |
| 25–50%      | Orange `#f97316` |
| < 25%       | Red `#ef4444`   |

### Milestone Events

- **1-min flash** — fires once at ≤60 sec (only if total ≥ 2 min): brief red background flash via CSS `@keyframes flash-1min`
- **15-sec flash** — fires once at ≤15 sec: continuous red screen flash until timer ends or user acts
- **End state** — freezes at `0:00`, plays double-pulse siren (square wave 380–760 Hz sweep), continuous flash until Restart or Back

### Audio (Web Audio API)

- **Heartbeat** (`playHeartbeat()`): 80 Hz sine pulse; only fires during the final 60 seconds. Louder during the final 15 sec.
- **Color burst**: louder heartbeat thump triggers on each color-state transition (`colorBurstTicks`)
- **Siren** (`playSiren()` / `stopSiren()`): synthesized on timer end; stopped when user taps Restart or Back

### Block View

10×3 grid (30 blocks). Each block ≈ 3.33% of total time. Blocks deplete left-to-right, top-to-bottom.

### Ring View

SVG arc depletes clockwise. Arc color tracks the color progression above.
