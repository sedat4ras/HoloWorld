# HoloWorld 🌍

A real-time holographic globe that spawns on your palm — controlled entirely through hand gestures. No hardware, no special equipment. Just a webcam and a browser.

https://github.com/user-attachments/assets/placeholder

## How It Works

| Gesture | Action |
|---|---|
| **Right hand snap** (thumb + middle finger) | Spawn the globe |
| **Right hand fist** | Despawn the globe |
| **Left pinch + drag** | Rotate the globe (trackball) |
| **Left pinch open/close** | Zoom in / out |
| **Left fist** | Pause auto-rotation |

## Tech Stack

- **[MediaPipe Holistic](https://google.github.io/mediapipe/solutions/holistic)** — real-time hand landmark detection (21 points per hand at 30fps)
- **[Three.js](https://threejs.org/)** — WebGL globe with lat/lon grid, orbital rings, and orbit particles
- **Canvas 2D** — webcam feed, spark effects, scan line, HUD overlays
- **Vanilla JS** — no framework, no build step

## Architecture

Two stacked canvases render simultaneously:

```
Canvas 2D (bottom)      Three.js WebGL (top, transparent)
─────────────────       ──────────────────────────────────
Mirrored webcam         Globe mesh + atmosphere halo
Spark burst on snap     Lat/lon grid lines
Left hand skeleton      Equatorial + orbital rings
Scan line overlay       Orbit particles (260 pts)
Palm connection beam    City data pins
HUD data panel          Spawn/despawn animation
```

MediaPipe runs asynchronously — the render loop consumes the latest landmarks each frame without blocking.

### Snap Detection

A two-phase state machine detects the snap gesture:
1. **LOAD** — thumb-to-middle-finger distance drops below `0.20 × palmSize`
2. **FIRE** — finger springs apart past `0.55 × palmSize` within 320ms with sufficient velocity

### Left Hand Control

Pinch uses hysteresis thresholds (0.36 enter / 0.45 exit) to prevent flickering. Fist detection uses a 6-frame ramp counter for stability. The skeleton is always rendered with gesture-reactive colors: **cyan** (idle) → **bright cyan** (grab) → **orange** (hold).

### Globe Positioning

The orthographic Three.js camera maps 1:1 to screen pixels. The globe anchors to the right palm center (smoothed with lerp factor 0.13) and floats `0.55 × palmSize` above it. Globe radius scales with palm size × zoom level, so it stays proportional regardless of camera distance.

## Run Locally

```bash
git clone https://github.com/sedat4ras/HoloWorld.git
cd HoloWorld
python3 -m http.server 8000
# Open http://localhost:8000
```

No install, no build. MediaPipe and Three.js load from CDN.

> **Note:** Camera access requires a secure context. `localhost` works fine; for remote access use `ngrok` or deploy to any static host (GitHub Pages, Netlify, Vercel).

## Performance Notes

- MediaPipe `modelComplexity: 1`, segmentation disabled for speed
- LUT tables for `Math.random()` and `Math.sin()` to avoid per-frame cost
- Orbit particles use a fixed `Float32Array` pool — no allocations in the render loop

## License

MIT
