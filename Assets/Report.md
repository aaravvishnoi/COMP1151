# COMP1151 Game Prototype – Report

**Student:** Aarav Vishnoi
**Game Title:** Desert Survivor
**Engine:** Unity 6.3.7f1 (WebGL)
**Asset Pack:** Kenney Desert Shooter Pack

---

## Part 1: Design Document

### Player Experience Goals

The two core experience goals targeted for this game are **Challenge** and **Drama**.

The initial vision was a generic "survival" game where the player avoids enemies. Through refinement, I narrowed this into a more specific and intentional experience: the player is a lone soldier stranded in a hostile desert, outnumbered and low on supplies. The goal was for every second of play to feel *tense but fair* — the player should always feel one bad decision away from death, but never feel like the outcome was outside their control.

**Challenge** was refined from "the game should be hard" into something more precise: the difficulty should come from *spatial reasoning and prioritisation*, not from reaction speed alone. Hazards patrol the world or actively chase the player, forcing the player to think about movement routes rather than just mashing keys. The tilemap obstacles narrow movement corridors, meaning the player cannot simply run in a straight line to escape.

**Drama** was refined from "there should be stakes" into *visible, felt consequences*. The health counter on-screen gives the player a constant reminder of their fragility. Each hit is not just a number drop — it's a step closer to the game-over screen. The sound design reinforces this: a distinct damage sound plays on each hit so the player *feels* every collision, even if they are watching the hazard rather than the UI.

### How Mechanical Choices Support the Goals

**Pickups** support both Challenge and Drama. They are not placed in safe, easy-to-reach locations — the spawner places them at random positions within the world, meaning the player must make a risk/reward decision: do I chase this pickup through an area patrolled by hazards, or do I stay safe and lose the score opportunity? This creates moment-to-moment drama around even a simple act of collection.

**Hazards** directly create Challenge. Two types of hazard exist: patrolling hazards that move along a fixed path (predictable, but forces path-planning) and chasing hazards that track the player's position (unpredictable, creates urgency). The combination means no single evasion strategy works for the whole game.

**The World** (tilemap + obstacles) shapes the Challenge by limiting movement. Rather than an open arena, the desert world uses obstacle walls and decorative props that create natural choke-points and corners. This gives hazards more power — they can trap the player — and makes the placement of pickups more interesting because the player must navigate around obstacles to reach them.

**Audio** supports Drama. The looping background music (from Incompetech) builds ambient tension without being distracting. Event-triggered sounds (pickup collected, damage taken) give the player immediate audio feedback that reinforces the cause-and-effect loop of the game.

### Playtesting Reflection

*[Complete after playtesting the built game — describe at least 2 playtests below.]*

**Playtest 1:**
- What worked: ...
- What didn't work: ...
- Specific improvement made: ...

**Playtest 2:**
- What worked: ...
- What didn't work: ...
- Specific improvement made: ...

The following aspects of the full player experience were evaluated during playtesting:
- Movement feel (is it responsive? too fast/slow?)
- Hazard difficulty (too easy to avoid, too hard to escape?)
- Pickup spawn frequency (too rare = boring, too common = trivial)
- UI clarity (can the player read health/score at a glance?)
- Audio feedback (does audio reinforce actions?)
- Game-over moment (is it clear why the player died?)

---

## Part 2: Implementation Document

### Feature 1 – WebGL Build

| | |
|---|---|
| **Build target** | WebGL |
| **Hosted at** | itch.io (see itch.io page for URL) |
| **Controls documented** | Yes — on itch.io page |
| **Third-party credits** | Listed on itch.io page (see Credits section below) |

The build was configured via *File → Build Settings → WebGL*. Both windowed and fullscreen modes work via the itch.io embed settings (set to *Full Window* with fullscreen button enabled).

---

### Feature 2 – Player Avatar

| | |
|---|---|
| **GameObject** | `Player` |
| **Script Machine** | `MovementScript` (ScriptGraphAsset) |
| **Sprite** | Kenney Desert Shooter Pack — character sprite |
| **Physics** | Rigidbody2D (Dynamic, GravityScale = 0, FreezeRotationZ) |
| **Collider** | CapsuleCollider2D |

**Unity features used:**
- `Rigidbody2D` for physics-based movement (velocity set each Update, inherently frame-rate independent)
- Input System (`COMP1151.inputactions`) — Player/Move action (Vector2), bound to WASD + Gamepad left stick
- Visual Script: `On Update` → `Input System` Read Value → `Vector2` → `Rigidbody2D.linearVelocity` × speed scalar

The player cannot rotate (FreezeRotationZ constraint on RB2D), preventing physics from spinning the sprite on collision.

---

### Feature 3 – World & Camera

| | |
|---|---|
| **Tilemap** | `Grid` → `Tilemap` (Ground layer), larger than one screen |
| **Decorative sprites** | `tilemap_packed_194 (2)`, `tilemap_packed_194 (3)`, `Sample A_0` (background) |
| **Camera** | `CinemachineBrain` on Main Camera; `CM camera` (CinemachineCamera + CinemachineFollow) |
| **Camera background** | Solid desert sand colour (not default blue) |
| **Camera size** | Orthographic size: 10, aspect ratio locked |

**Unity features used:**
- Cinemachine 3.1.6: `CinemachineCamera` with `CinemachineFollow` extension targeting the Player's Transform
- `CinemachineBrain` on the Main Camera drives the output
- `Camera.orthographic = true`, background colour set to a warm sand tone

---

### Feature 4 – Obstacles

| | |
|---|---|
| **Obstacle type** | Tilemap layer (`Obstacles` Tilemap child of `Grid`) |
| **Collider** | Tilemap Collider 2D + Composite Collider 2D |
| **Physics Layer** | `Obstacle` layer |
| **Layer Collision Matrix** | Player ↔ Obstacle = enabled; Pickup ↔ Obstacle = disabled; Hazard ↔ Obstacle = enabled |

**Unity features used:**
- `Tilemap Collider 2D` (Used by Composite = true) + `Composite Collider 2D` for efficient static collision
- Physics Layers: `Player` (Layer 6), `Obstacle` (Layer 7), `Pickup` (Layer 8), `Hazard` (Layer 9)
- Layer Collision Matrix configured in *Project Settings → Physics 2D*

---

### Feature 5 – Pickups

| | |
|---|---|
| **GameObject (Prefab)** | `Pickup` prefab in `Assets/Prefabs/` |
| **Script Machine** | `PickupLogic` (on Pickup prefab) |
| **Spawner** | `PickupSpawner` GameObject — Script Machine: `PickupSpawner` |
| **Sprite** | Kenney Desert Shooter Pack — ammo/item sprite |

**Unity features used:**
- Trigger Collider (`CircleCollider2D`, IsTrigger = true) on Pickup prefab
- `On Trigger Enter 2D` event in Visual Script — checks colliding object is Player, increments score variable, destroys self
- Spawner uses `On Timer` / `Wait` node with random position via `Random.insideUnitCircle` to instantiate Pickup prefab at varied times

---

### Feature 6 – Hazards

| | |
|---|---|
| **GameObject (Prefab)** | `Hazard` prefab in `Assets/Prefabs/` |
| **Script Machine** | `HazardMovement` (on Hazard prefab) |
| **Movement type** | Chase (moves toward Player position each Update) |
| **Sprite** | Kenney Desert Shooter Pack — enemy sprite |

**Unity features used:**
- `Rigidbody2D` on Hazard — velocity set toward Player each frame (via `Vector2.MoveTowards` or direction × speed)
- `On Trigger Enter 2D` — detects Player collision, calls custom event to decrease health
- Multiple prefab instances placed in the scene (or spawned by a separate Hazard spawner)

---

### Feature 7 – UI

| | |
|---|---|
| **Canvas** | `UICanvas` (Screen Space – Overlay) |
| **Score Text** | `ScoreText` — anchored top-left |
| **Health Text** | `HealthText` — anchored top-right |
| **Game-Over Panel** | `GameOverPanel` — centered, inactive by default |
| **Script Machine** | `UIManager` (on `UICanvas`) |

**Unity features used:**
- `TextMeshProUGUI` for Score and Health display
- Scene Variables: `score` (int), `health` (int, starts at 3)
- `UIManager` Visual Script: listens for `ScoreChanged` and `HealthChanged` custom events, updates Text components; when `health ≤ 0`, activates `GameOverPanel` and pauses game (`Time.timeScale = 0`)
- Anchors set in RectTransform so UI stays in corners at any resolution

---

### Feature 8 – Audio

| | |
|---|---|
| **BGM** | `AudioManager` GameObject — AudioSource, loop = true, Incompetech track |
| **Spatial source** | `SpatialAudioSource` — 3D AudioSource placed in world |
| **SFX** | `PickupSFX`, `DamageSFX` — triggered via Visual Script custom events |

**Unity features used:**
- `AudioSource.Play()` called from Visual Script `On Custom Event` nodes
- BGM: 2D AudioSource (Spatial Blend = 0), loop enabled, plays on Awake
- Spatial source: Spatial Blend = 1.0 (fully 3D), Min/Max Distance configured, plays ambient world sound
- Pickup and damage sounds: short clips played via `AudioSource.PlayOneShot` nodes in PickupLogic and HazardMovement Visual Scripts

---

## Credits / Third-Party Assets

| Asset | Author | URL | License |
|---|---|---|---|
| Desert Shooter Pack | Kenney | https://kenney.nl/assets/desert-shooter-pack | CC0 1.0 |
| Roguelike Characters 2 | Kenney | https://kenney.nl/assets/roguelike-characters-2 | CC0 1.0 |
| Background Music | Kevin MacLeod (Incompetech) | https://incompetech.com | CC BY 3.0 |
| SFX | Pixabay | https://pixabay.com/sound-effects/ | Pixabay Content License |
| Font | Font Library | https://fontlibrary.org | (specify font and license) |
| Super Tiled2Unity | Seanba | https://github.com/Seanba/SuperTiled2Unity | MIT |
| Unity Cinemachine | Unity Technologies | https://unity.com/unity/features/editor/art-and-design/cinemachine | Unity Package License |
