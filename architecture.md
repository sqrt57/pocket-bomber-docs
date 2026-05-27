# Pocket Bomber — Architecture

## Stack

Flutter app shell → `GameWidget` → Flame `FlameGame` subclass → Flame components.

Flutter handles app lifecycle and orientation lock. Flame owns the game loop, rendering, and input.

## Entry point

```
main.dart
  └── PocketBomberApp (MaterialApp)
        └── GameScreen (StatelessWidget)
              └── GameWidget(game: PocketBomberGame)
```

## Component hierarchy

```
PocketBomberGame (FlameGame)
  ├── GridComponent          — tile map render + tap hit-testing
  ├── PlayerComponent        — player sprite, grid position, power-up state
  ├── EnemyComponent (×0–3)  — random-wander AI, grid position
  ├── BombComponent (×0–N)   — countdown timer, triggers ExplosionComponent
  ├── ExplosionComponent     — blast rays in 4 cardinal directions
  └── PowerUpComponent       — revealed on soft-wall destruction, collected on contact
```

## Coordinate system

Grid cells are the primary coordinate unit. Each cell is 40dp × 40dp. Components store their position as `(col, row)` integers and compute pixel position as `(col * tileSize, hudHeight + row * tileSize)`.

Pixel ↔ grid conversion lives in `grid.dart` and is used by all components.

## Game loop flow

Each frame `PocketBomberGame.update(dt)` cascades into component `update(dt)` calls:

1. **PlayerComponent** — applies queued move if tile is clear; checks power-up/enemy/explosion overlap
2. **EnemyComponent** — advances wander timer; picks random valid direction; checks explosion overlap
3. **BombComponent** — counts down; on zero: spawns `ExplosionComponent`, destroys self
4. **ExplosionComponent** — short-lived (≈0.5s); on spawn: destroys soft walls, triggers chained bombs, kills players/enemies in range; destroys self after duration

## Input

Taps are handled in `PocketBomberGame.onTapDown`. Tap position is converted to grid cell. Relative to player:

- Same cell → place bomb
- Adjacent cell (cardinal) → queue player move
- Non-adjacent or hard wall → ignore

## State transitions

```
playing  ──(all 10 enemies killed)──► exit tile appears
playing  ──(player reaches exit)────► level complete
playing  ──(player hit)─────────────► game over → restart
```

No persistent state between sessions in the demo. Score and kill count live on `PocketBomberGame` and are read by the HUD.

## Key files

| File | Purpose |
|---|---|
| `lib/main.dart` | App entry, orientation lock |
| `lib/game/pocket_bomber_game.dart` | FlameGame root, input handler, game state |
| `lib/game/grid.dart` | Tile map data, cell ↔ pixel conversion |
| `lib/game/grid_component.dart` | Grid renderer |
| `lib/game/player.dart` | Player component |
| `lib/game/enemy.dart` | Enemy AI component |
| `lib/game/bomb.dart` | Bomb countdown + explosion trigger |
| `lib/game/explosion.dart` | Blast ray component |
| `lib/game/powerup.dart` | Power-up component |
| `lib/screens/game_screen.dart` | GameWidget wrapper |
| `lib/screens/game_over_screen.dart` | Game over overlay |
