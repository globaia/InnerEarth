# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

InnerEarth is an immersive 3D visualization where the user stands INSIDE a sphere, looking at Earth's surface projected onto its inner surface using an azimuthal equidistant projection.

- **Nadir** (user's location) = bottom of sphere (-Y axis)
- **Zenith** (antipode) = top of sphere (+Y axis)
- Users can walk using WASD/arrow keys, with the world rotating around them

Live demo: https://globaia.github.io/InnerEarth/

## Development Commands

```bash
# Start local development server
python3 -m http.server 8000
# Open http://localhost:8000

# The entire app is in index.html - no build step required
```

## Architecture

### Single-File Application
The entire application is contained in `index.html` (~167KB), including:
- HTML structure and CSS styles
- All JavaScript (Three.js WebGL rendering)
- UI controls and event handlers

### Key Coordinate System
The `geoToSphere(lat, lon)` function is the core transformation:
1. Calculate angular distance and azimuth from nadir to target point
2. Map to sphere position where nadir is at -Y, antipode at +Y

```
Geographic (lat,lon) → Angular distance + Azimuth → Sphere position (x,y,z)
```

### World Rotation for Walking
All sphere content is in `worldGroup`, which rotates when walking. This provides smooth movement without rebuilding geometry every frame. The `buildNadir` tracks the nadir at geometry build time, while `nadir` tracks current position.

### Layer System
Multiple visual layers share the same projection:
- Satellite imagery (equirectangular texture mapped to sphere)
- Coastlines (Natural Earth GeoJSON vectors)
- Graticule grid (latitude/longitude lines)
- City markers and labels
- 3D walking character (FBX model with animations)

## Critical Implementation Details

### Satellite/Vector Alignment
The satellite sphere MUST use the same `geoToSphere()` projection as coastlines. Using standard Three.js SphereGeometry with rotation will NOT align correctly.

### UV Mapping for Satellite Texture
For equirectangular textures:
- Latitude iterates +90 to -90 (north to south)
- UV v=0 is north (top of texture), v=1 is south (bottom)
- Use `THREE.DoubleSide` to ensure visibility from inside sphere

### Z-Fighting Prevention
Place satellite sphere at slightly larger radius (1.002x) than vector lines to render behind them without flickering.

## GeoQuiz Race Module

A geography quiz game mode where players walk toward answer locations on the globe.

### Entry Points
- "Play GeoQuiz Race" button in the info panel (top-left)
- `P` key to start, `Escape` to cancel mid-quiz

### State Machine
`IDLE → SHOWING_QUESTION → WALKING → RESULT → STATS → LEADERBOARD`

### Key Functions
- `startQuiz()` — Saves speed, locks to 10k km/h, hides UI, shuffles questions
- `showQuestion(index)` — Displays question text, creates 3 answer spheres via `geoToSphere()`
- `createAnswerSphere()` — Canvas-based sprite with colored glow and label
- `checkQuizCollision()` — Runs each frame during WALKING; uses `angularDistance()` with 5° threshold
- `showResult()` / `showStats()` / `showLeaderboard()` — UI state transitions
- `endQuiz()` — Restores speed, slider, UI visibility, cleans up spheres
- `rebuildQuizSpheres()` — Called from `rebuildAll()` to recreate spheres after world rebuild

### Integration Points
- `animate()` — Calls `checkQuizCollision()`, `updateQuizTimer()`, `updateQuizPulse()` when walking
- `updateWalking()` — Speed lock: `if (quizState !== 'IDLE') walkSpeedKmh = QUIZ_SPEED_KMH`
- Speed slider — Guarded: `if (quizState !== 'IDLE') return`
- `keydown` handler — `P` to start, `Escape` to cancel
- `rebuildAll()` — Calls `rebuildQuizSpheres()` if quiz is active

### Constants
- `QUIZ_SPEED_KMH = 10000` (locked during quiz)
- `QUIZ_DETECTION_DEGREES = 5` (~556 km radius)
- `QUIZ_NUM_QUESTIONS = 10` (selected from 20-question bank)
- `QUIZ_RESULT_DISPLAY_MS = 2500`

### localStorage
- Key: `innerearth_geoquiz_leaderboard` — Top-10 scores as JSON array

### Design Document
See `Text/GeoQuiz_Race_Design.md` for full game design details.

## External Dependencies (loaded from CDN)
- Three.js r128
- FBXLoader for 3D character
- Natural Earth coastline GeoJSON (fetched at runtime)
- OpenStreetMap Nominatim API (location search)

## Assets
- `assets/earth/earth_4k.jpg` - Satellite texture
- `assets/models/character.fbx` - 3D character model (20MB)
- `assets/models/walking.fbx`, `idle.fbx` - Character animations
