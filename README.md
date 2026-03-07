# Oxford Discover Vocab Master 🎮

A single-file, HTML5 vocabulary game built for the **Oxford Discover** series.
Drop it into any folder alongside your image and audio assets — no build tools, no server, no dependencies.

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Vanilla](https://img.shields.io/badge/No_Framework-Vanilla_JS-informational?style=flat)

---

## Table of Contents

- [Features](#features)
- [Demo](#demo)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Changing the Vocabulary](#changing-the-vocabulary)
- [Adding a New Mission Type](#adding-a-new-mission-type)
- [Cloud Backend](#cloud-backend)
- [Architecture Overview](#architecture-overview)
- [Asset Preloading](#asset-preloading)
- [Browser Support](#browser-support)
- [License](#license)

---

## Features

| Feature | Details |
|---|---|
| **Centralized Config** | One `GAME_CONFIG` object controls all words, difficulty, emoji fallbacks, and storage keys — change the whole game without touching any logic |
| **Modular Architecture** | Seven decoupled layers: `GameEngine`, `UIRenderer`, `MissionRegistry`, `CloudManager`, `AssetLoader`, `GameState`, `GameController` |
| **Asset Preloading** | All images and audio are cached before play starts; a live `LOADING ASSETS… 67%` progress counter is shown on the splash screen |
| **Graceful Fallbacks** | Broken image paths auto-swap to emoji; broken audio paths fall back to the Web Speech API (TTS) |
| **Lego-Style UI** | Four-color brick tile system with gloss, studs, press animations, and a full CSS custom-property neon theme |
| **Visual Feedback System** | Particle burst, glitch flash, dual-coding feedback overlay, combo badge, and urgent timer pulse — all pure CSS animations |
| **Three Mission Types** | Visual Match (image → word), Sonic Match (audio → image), Spell Match (audio → word) |
| **Extensible Missions** | `MissionRegistry.register()` lets you add new question formats with one isolated call — the core engine is never touched |
| **Cloud-Ready** | `CloudManager` wraps all persistence; connect a real API by filling two fields and uncommenting four lines |
| **Leaderboard & Ranks** | Per-unit local leaderboard (top 20), Hall of Fame screen, Personal Best banner, S/A/B/C rank system, combo streak badges |

---

## Demo

Open `index.html` directly in any modern browser — no web server required.

```bash
# macOS
open index.html

# Windows
start index.html

# Linux
xdg-open index.html
```

---

## Getting Started

1. **Clone or download** this repository.
2. Place your image files in `images/` and audio files in `audio/`.
3. Open `index.html` in a browser.
4. Edit `GAME_CONFIG` at the top of the `<script>` section to swap in your own vocabulary (see [Changing the Vocabulary](#changing-the-vocabulary)).

```
Oxford Discover Vocab Master_Project/
├── index.html          ← entire game (HTML + CSS + JS in one file)
├── README.md
├── images/
│   ├── chick.jpg
│   ├── crab.jpg
│   └── ...             ← one image per vocabulary word
└── audio/
    ├── chick.mp3
    ├── crab.mp3
    └── ...             ← one audio pronunciation file per word
```

> Asset paths are **relative**. Keep `images/` and `audio/` next to `index.html` and the game works with no configuration.

---

## Project Structure

The entire game lives in **`index.html`** and is organised into seven logical layers inside the `<script>` tag:

```
GAME_CONFIG        ← all content & settings (edit this to change the game)
CloudManager       ← data persistence (local + cloud-ready)
AssetLoader        ← preloads img/audio before game starts
GameEngine         ← pure logic — zero DOM access
MissionRegistry    ← open registry of question-type definitions
UIRenderer         ← all DOM manipulation — zero game logic
GameState          ← mutable runtime values
GameController     ← thin orchestration glue
```

---

## Changing the Vocabulary

Open `index.html` and find `GAME_CONFIG` at the very top of the `<script>` section. Everything you need to customise is here.

### 1. Update unit metadata

```js
META: {
  TITLE:          "Vocab Master",
  SUBTITLE:       "Unit 6 · Ocean Life",    // shown on Splash & Hall of Fame
  UNIT_TAG:       "Unit 6",
  STORAGE_SUFFIX: "unit6_v1",               // ← CHANGE THIS for every new unit
},
```

> ⚠️ **Always change `STORAGE_SUFFIX`** when deploying a new unit. This gives each unit its own isolated leaderboard in `localStorage` so Unit 5 and Unit 6 scores never mix.

### 2. Replace the word list

```js
WORDS: [
  { word: "Dolphin",  img: "images/dolphin.jpg",  audio: "audio/dolphin.mp3",  sfx: "" },
  { word: "Octopus",  img: "images/octopus.jpg",  audio: "audio/octopus.mp3",  sfx: "" },
  { word: "Seahorse", img: "images/seahorse.jpg", audio: "audio/seahorse.mp3", sfx: "" },
  // add as many words as needed
],
```

| Field | Required | Notes |
|---|---|---|
| `word` | ✅ | Display text; also used as the TTS utterance when audio is missing |
| `img` | No | Relative path to image file; leave `""` to use emoji fallback |
| `audio` | No | Relative path to audio file; leave `""` to use TTS fallback |
| `sfx` | No | Reserved for future per-word sound effects |

### 3. Update the emoji fallback map

Add an entry for every new word so missing images still look polished:

```js
EMOJI: {
  dolphin:  "🐬",
  octopus:  "🐙",
  seahorse: "🌊",
  // keys must be lowercase strings matching the word field
},
```

### 4. Tune difficulty (optional)

```js
SETTINGS: {
  TIMER_SEC:   12,    // seconds per question (default: 15)
  COMBO_MS:    2500,  // max ms for an answer to count as a combo (default: 3000)
  MAX_LB_SIZE: 20,    // max leaderboard entries to keep in storage
  POINTS: { fast: 150, mid: 120, slow: 100, late: 80 },
  RANK: {
    S: { minAcc: 90, maxAvgSec: 4 },
    A: { minAcc: 75, maxAvgSec: 6 },
    B: { minAcc: 55, maxAvgSec: Infinity },
    // anything below B thresholds → C rank
  },
},
```

---

## Adding a New Mission Type

The game ships with `VISUAL_MATCH`, `SONIC_MATCH`, and `SPELL_MATCH`. Adding a fourth type requires **one isolated call** — no engine code changes needed.

```js
// Step 1 — Register the new type (add after the existing register() calls)
MissionRegistry.register("DEFINITION_MATCH", {
  label: "📖  Read the definition — which word fits?",

  buildMain(target) {
    // Return a DOM node to display in the #game-main area
    const box       = document.createElement("div");
    box.className   = "main-img-wrap";
    box.textContent = target.definition; // add a `definition` field to each WORD entry
    return box;
  },

  buildTileContent(item) {
    // Return an array of DOM nodes to place inside each answer tile
    const lbl       = document.createElement("div");
    lbl.className   = "tile-text";
    lbl.textContent = item.word;
    return [lbl];
  },

  onLoad(target) {
    // Called automatically after the mission renders (e.g. auto-read the definition)
    // Snd.speak(target.definition);
  },
});

// Step 2 — Include it in GameEngine.buildMissions()
GAME_CONFIG.WORDS.forEach(item =>
  pool.push({ type: "DEFINITION_MATCH", target: item })
);
```

---

## Cloud Backend

`CloudManager` is the **sole owner** of all data reads and writes. By default it uses `localStorage`. Connecting a real API requires two values and four uncommented lines — nothing else in the codebase changes.

### Step 1 — Set your credentials

```js
const CloudManager = {
  API_URL: "https://api.your-backend.com",  // ← fill in
  API_KEY: "your-secret-key",               // ← fill in
  ...
};
```

### Step 2 — Uncomment the fetch stubs

Inside `saveScore()`, uncomment the POST block:

```js
fetch(this.API_URL + "/scores", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${this.API_KEY}`,
  },
  body: JSON.stringify(entry),
}).catch(err => console.warn("Cloud sync failed:", err));
```

Inside `fetchLeaderboard()`, uncomment the GET block:

```js
return fetch(this.API_URL + "/leaderboard", {
  headers: { "Authorization": `Bearer ${this.API_KEY}` },
})
  .then(r => r.json())
  .catch(() => cached); // falls back to local cache on any network failure
```

> The rest of the game already calls `CloudManager.fetchLeaderboard()` via `.then()` and handles the Promise whether it resolves locally or remotely — no further changes needed.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                     index.html                       │
│                                                      │
│  ┌─────────────┐   reads    ┌──────────────────┐    │
│  │ GAME_CONFIG │ ◄──────── │   GameController  │    │
│  └─────────────┘            │  (orchestration)  │    │
│                             └────────┬──────────┘    │
│  ┌─────────────┐                     │               │
│  │ CloudManager│ ◄── save/fetch ─────┤               │
│  └─────────────┘                     │               │
│                                      ├── logic ───►  │
│  ┌─────────────┐                ┌────┴──────────┐    │
│  │ AssetLoader │ ── preloads ──►│  GameEngine   │    │
│  └─────────────┘                └───────────────┘    │
│                                      │               │
│  ┌──────────────────┐                ├── render ──►  │
│  │ MissionRegistry  │ ◄── get() ─────┤  ┌──────────┐ │
│  │  VISUAL_MATCH    │                └─►│UIRenderer│ │
│  │  SONIC_MATCH     │                   └──────────┘ │
│  │  SPELL_MATCH     │                                │
│  │  + your types    │                                │
│  └──────────────────┘                                │
└─────────────────────────────────────────────────────┘
```

---

## Asset Preloading

On page load, `AssetLoader.preload()` iterates every entry in `GAME_CONFIG.WORDS` and creates a hidden `Image` and `Audio` element for each path.

```
Page opens
  → PLAY button disabled
  → "LOADING ASSETS… 0%" displayed on splash screen
  → img.onload  / audio.canplaythrough events tick the counter
  → "LOADING ASSETS… 100%"
  → "✓ READY!" shown, PLAY button enabled
```

**Resilience mechanisms:**

| Scenario | Behaviour |
|---|---|
| Image returns 404 | Counted as loaded; emoji shown at render time via `img.onerror` |
| Audio returns 404 | Counted as loaded; TTS used at playback time via `audio.onerror` |
| Audio never fires `canplaythrough` | Per-file 3-second timeout triggers (common on mobile) |
| Entire batch stalls | Global 8-second safety net unblocks the splash regardless |

---

## Browser Support

| Feature used | Chrome | Firefox | Safari | Edge |
|---|---|---|---|---|
| CSS custom properties & animations | 49+ | 31+ | 9.1+ | 16+ |
| Web Audio API (sound effects) | 35+ | 25+ | 14.1+ | 79+ |
| Web Speech API (TTS fallback) | 33+ | 49+ | 7+ | 14+ |
| `Element.animate()` (particles) | 36+ | 48+ | 13.1+ | 79+ |
| `localStorage` | All modern | All modern | All modern | All modern |

> The game is fully playable on modern **mobile browsers** (iOS Safari 14+, Chrome for Android).
> On older browsers, sound effects degrade gracefully to silence and TTS degrades to a beep — gameplay is never blocked.

---

## License

For educational use within the Oxford Discover series.
