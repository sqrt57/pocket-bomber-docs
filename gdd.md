# Pocket Bomber — Planning Document

## Tech Stack Decision

### Platform
- **Flutter + Flame**
- Portrait mode only
- **Active targets:** Android (release), Web (dev iteration)
- **Deferred:** iOS — needs a Mac with Xcode; implement when ready to ship
- **Dropped:** macOS, Windows, Linux — removed from project

### Why Flutter + Flame
- Code-first, no visual editor — Claude Code can drive almost everything
- Flutter handles mobile plumbing (app lifecycle, permissions, stores)
- Flame handles game loop, sprites, collisions, scenes
- Hot reload for fast iteration
- Dart is simple and Claude Code knows it well

### Flutter Platform Support
Flutter compiles to 6 platforms from one codebase:
- iOS, Android, Web, macOS, Windows, Linux

### Flutter vs Kotlin vs KMP (2026 context)
- **Kotlin** — sole recommended language for native Android, 80% Google Play market share, Android-only
- **Flutter** — Google's cross-platform framework, "Production Era" with Impeller engine stable, overtook React Native in usage
- **KMP (Kotlin Multiplatform)** — fastest growing, shares business logic, keeps native UI; used by Netflix, Airbnb, Cash App
- For a casual 2D mobile game targeting iOS + Android → Flutter + Flame wins

### Dart Language Status (2026)
- Niche but healthy — almost exclusively used with Flutter
- Flutter has overtaken React Native in frequency of use
- Not a general-purpose career language; learn it as a Flutter vehicle
- Consistently ranks in Top 20 programming languages

---

## Game Spec

### Grid
- **Size:** 9 columns × 11 rows
- **Tile size:** 40dp
- **Total grid:** 360dp × 440dp (fits all phones in portrait)
- **HUD bar:** ~50dp at top (score, bomb count, blast radius)
- No scrolling — entire grid visible at all times
- Odd dimensions both ways → classic hard wall pattern works

### Tile Types
- **Hard walls** — indestructible, form the maze (at even col+row intersections)
- **Soft walls** — destructible, may hide power-ups
- **Floor** — walkable
- **Exit** — appears after all enemies are killed

### Controls
- **Tap above player** → move up
- **Tap below player** → move down
- **Tap left of player** → move left
- **Tap right of player** → move right
- **Tap ON player** → place bomb
- No on-screen buttons needed — whole grid is the input surface

### Bombs
- Infinite supply (no ammo limit)
- Player can have **1 bomb on grid at a time** (upgradeable)
- **3 second countdown**
- Blast in **4 cardinal directions only** (no diagonals):
  ```
  . 💥 .
  💥 💣 💥
  . 💥 .
  ```
- **Starting blast radius:** 1 tile in each direction
- **Chain explosions:** blast hitting another bomb triggers it early
- Blast stops at hard walls, destroys soft walls

### Enemies
- **3 simultaneous** on grid at any time
- **10 total** to kill (respawn up to 3 simultaneous as you kill them)
- **Behavior:** random wander (tutorial level)
- Level 1 is the demo — no difficulty scaling needed yet

### Win Condition
- Kill all 10 enemies → exit tile appears
- Player reaches exit → level complete

### Lose Condition
- One hit = dead (from explosion or enemy touch)
- Restart level
- No lives system

### Power-ups (hidden under soft walls, revealed on destruction)
- 💣 **Extra bomb** — allows one more simultaneous bomb on grid
- 🔥 **Blast radius** — extends explosion range by 1 tile
- 👟 **Speed** — increases player movement speed

### Scoring
- **100 points** per enemy killed
- Displayed in HUD

### Levels
- **1 level** (demo)
- Expand later based on playtest feedback

### Art
- **Start with geometric shapes** (colored rectangles/circles)
- Add proper sprites once mechanics are working
- AI-generated pixel art sprites available as next step

### Sound
- None for demo

---

## Implementation Plan

### Stage 0: Infrastructure Setup (1–2h)

**0a: Dev Environment (30min–1h)**
- Install Flutter SDK
- Install Dart
- Install Android Studio + emulator OR physical device setup
- Install Flutter/Dart extensions for VS Code
- Verify with `flutter doctor`

**0b: Project Bootstrap (15–30min)**
- `flutter create bomberman`
- Add Flame dependency to `pubspec.yaml`
- Set up folder structure (`game/`, `screens/`, `components/`)
- Configure portrait-only orientation in `AndroidManifest.xml` and `Info.plist`
- Git init + first commit
- Verify app runs on emulator/device

**0c: AI Documentation Repo (30min)**
- Create `docs/` folder in project root
- Create `CLAUDE.md` — main context file Claude Code reads automatically every session
- Create `docs/architecture.md` — component relationships, game loop flow
- Create `docs/gamedesign.md` — full game spec
- Commit to git

---

### Stage 1: Grid Rendering (1–2h)
- Draw 9×11 grid with hard walls, soft walls, floor
- Classic hard wall pattern (hard walls on even col+row intersections)
- Tap detection on grid

### Stage 2: Player Movement (1–2h)
- Player component on grid
- Tap direction logic → move one tile
- Collision with hard and soft walls
- Smooth animation between tiles

### Stage 3: Bombs & Explosions (2–3h)
- Place bomb on tap-self
- 3 second countdown
- Explosion in 4 directions, 1 tile radius
- Stops at hard walls, destroys soft walls
- Chain explosion logic
- One bomb at a time enforcement

### Stage 4: Enemies (2–3h)
- Enemy component with random wander AI
- Respawn up to 3 simultaneous, 10 total counter
- Enemy dies from explosion
- Player dies from explosion or enemy touch

### Stage 5: Power-ups (1–2h)
- Hidden under random soft walls
- Reveal on wall destruction
- Extra bomb, blast radius, speed pickups
- Apply effect on player contact

### Stage 6: Win / Lose / Score (1h)
- Score counter (+100 per kill)
- Kill counter tracking 10 total
- Exit tile appears after 10 kills
- Win screen on exit reached
- Death screen with restart

### Stage 7: HUD (1h)
- Top bar: score, bomb count, blast radius
- Game over / level complete overlays

### Stage 8: Polish & Bugfix (2–3h)
- Edge cases (chain explosions, player in blast, spawn safety)
- Balancing soft wall density and power-up frequency
- Playtest and tweak

---

## Time Estimate

| Stage | Description | Time |
|---|---|---|
| 0a | Flutter + Android Studio setup | 30min–1h |
| 0b | Project bootstrap | 15–30min |
| 0c | AI documentation repo | 30min |
| 1 | Grid rendering | 1–2h |
| 2 | Player movement | 1–2h |
| 3 | Bombs & explosions | 2–3h |
| 4 | Enemies | 2–3h |
| 5 | Power-ups | 1–2h |
| 6 | Win/lose/score | 1h |
| 7 | HUD | 1h |
| 8 | Polish & bugfix | 2–3h |
| **Total** | | **~12–18h** |

---

## Folder Structure
```
bomberman/
  lib/
    main.dart
    game/
      bomberman_game.dart   # FlameGame root
      grid.dart             # tile map logic
      player.dart           # player component
      enemy.dart            # enemy AI
      bomb.dart             # countdown + explosion
      explosion.dart        # blast ray component
      powerup.dart
    screens/
      menu_screen.dart
      game_screen.dart
      game_over_screen.dart
  docs/
    CLAUDE.md               # Claude Code context file
    architecture.md
    gamedesign.md
```

---

## Next Steps
1. Draft `CLAUDE.md` content for Claude Code sessions
2. Start Stage 0a — Flutter environment setup
3. Each stage = one focused Claude Code session with clear goal and testable result

---

## Name Candidates

**Code name:** Pocket Bomber

**Marketing name variants (to evaluate later):**

2-word:
- Blast Radius
- Fuse & Flee
- Blast Zone
- Kill Radius
- Ground Zero
- Shock Wave
- Micro Bomber
- Tiny Blaster
- Handheld Havoc
- Drop & Dash
- Plant & Panic
- Light & Run

3-word:
- Light the Fuse
- Tick Tick Boom
- Short Fuse Runner
- One Tile Radius
- Plant and Pray
- Drop the Bomb
