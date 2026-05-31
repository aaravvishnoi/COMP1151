# COMP1151 Game Prototype – Report

**Student:** Aarav Vishnoi
**Game Title:** Desert Survivor
**Engine:** Unity 6 (6000.3.7f1), WebGL build
**Asset Packs:** Kenney Desert Shooter Pack, Kenney Roguelike Characters 2

---

## Part 1: Design Document

### Player Experience Goals

The two core experience goals targeted for this game are **Challenge** and **Drama**.

The initial vision was a generic "survival" game where the player avoids enemies. Through refinement, I narrowed this into a more specific and intentional experience: the player is a lone soldier stranded in a hostile desert, outnumbered and low on supplies. The goal was for every second of play to feel *tense but fair* — the player should always feel one bad decision away from death, but never feel like the outcome was outside their control.

**Challenge** was refined from "the game should be hard" into something more precise: the difficulty should come from *spatial reasoning and prioritisation*, not from reaction speed alone. Hazards actively chase the player, and they arrive in escalating waves, forcing the player to think about movement routes and target priority rather than just mashing keys. The buildings and obstacles narrow movement corridors, meaning the player cannot simply run in a straight line to escape.

**Drama** was refined from "there should be stakes" into *visible, felt consequences*. The health counter on the HUD gives the player a constant reminder of their fragility — each hit visibly drops it and brings the game-over screen one step closer. The limited magazine and reload window add a second layer of stakes: running dry at the wrong moment is its own small crisis. Audio cues on firing and collecting pickups keep the player informed of their actions even while their eyes are on the hazards rather than the UI.

### How Mechanical Choices Support the Goals

**Pickups** support both Challenge and Drama. They are not placed in safe, easy-to-reach locations — a spawner places them at random positions across the whole map, meaning the player must make a risk/reward decision: do I chase this pickup through an area dense with hazards, or do I stay safe and lose the score opportunity? This creates moment-to-moment drama around even a simple act of collection.

**Hazards and the wave system** directly create Challenge. Hazards chase the player by tracking their position each physics step, and they spawn in escalating waves: each wave increases both the *number* of hazards and the *health* of each one (so later enemies take more shots to kill). This produces a clear, rising difficulty curve and forces the player to keep moving and to prioritise which threats to deal with first — no single evasion strategy works for the whole game.

**Shooting** gives the player agency within the Challenge. A gun must first be found in the world; once collected, the player fires toward the cursor, but a six-round magazine and a reload delay mean firepower is a managed resource rather than unlimited, reinforcing the "low on supplies" fantasy.

**The World** (multi-layer tilemap + buildings) shapes the Challenge by limiting movement. Rather than an open arena, the desert town uses solid buildings and boundary walls that create natural choke-points and corners, giving hazards more power to trap the player and making pickup placement more interesting because the player must navigate around obstacles to reach them.

**Audio** supports Drama. Looping background music builds ambient tension without being distracting, and event-triggered sounds (firing the gun, collecting a pickup) give the player immediate audio feedback that reinforces the cause-and-effect loop of the game.

### Playtesting Reflection

Playtesting was carried out iteratively on the built game, with each round focused on whether the **Challenge** and **Drama** goals were actually being felt by the player rather than just present in the design. Two of the most significant rounds are described below.

**Playtest 1 — Combat fairness and aiming (early build):**
- **What worked:** The wave structure landed well. Starting with three weak hazards and scaling both the count and their health each wave gave a clear, readable difficulty ramp — early waves taught the controls, later waves forced the player to keep moving and prioritise targets. Shooting felt satisfying and the magazine/reload loop created good "low on ammo" tension, directly supporting the Challenge goal.
- **What didn't work:** Two issues broke the sense of *fairness* that Challenge depends on. First, hazards appeared to "vanish" when the player moved away and "pop back" when they returned — they were spawning at fixed world coordinates near the origin rather than near the player, so when the player roamed they were left fighting nothing, then suddenly swarmed. Second, bullets drifted in the player's *movement* direction instead of flying to the cursor: holding **A** sent shots left regardless of aim, which felt broken and made combat feel out of the player's control (the opposite of the "tense but fair" goal).
- **Specific improvement made:** Hazard spawning was rewritten to place enemies *relative to the player's current position*, clamped to the map bounds, so encounters stay consistent wherever the player goes. The bullet aim was fixed to compute its direction from the player's **current** position (it had been using the bullet's spawn position, which lagged one frame behind the camera, leaking movement into the shot). After the fix, shots reliably travel to the cursor, restoring player agency.

**Playtest 2 — Readability and the play space (mid build):**
- **What worked:** With spawning fixed, the moment-to-moment combat became consistent and the risk/reward of chasing pickups through hazard-dense areas came through as intended (Drama). Audio feedback on shooting and pickup collection reinforced the cause-and-effect loop — testers reacted to the sounds without watching the UI.
- **What didn't work:** Two clarity problems surfaced. The HUD showed bare numbers (e.g. "5", "0", "6") with no labels, so testers couldn't tell health from ammo from score at a glance during combat — they were losing track of how close to death they were, which undercut the Drama goal of *felt* fragility. Separately, the play space felt flat and unfair: the player could wander off the edge into empty void, and pickups were clustered in the centre, so most of the map was dead space with nothing to do.
- **Specific improvement made:** The HUD was redesigned into a labelled, colour-coded panel (**WAVE / HEALTH / SCORE / AMMO**), each value sitting next to a coloured label so state is readable at a glance. The world was built out into a proper desert map with buildings (solid colliders), props and varied ground, and invisible boundary walls were added so the player is kept inside the play space. Pickups were changed to spawn at random positions across the *whole* map, so every area now offers a reason to move into it — strengthening the spatial-reasoning Challenge.

**Aspects evaluated across playtesting:**

| Aspect | Finding | Outcome |
|---|---|---|
| Movement feel | Responsive; speed felt right once the camera followed smoothly | Kept as-is |
| Hazard difficulty | Good ramp via wave scaling, but spawn location felt random/unfair | Fixed with player-relative, map-clamped spawning |
| Aiming | Bullets followed movement, not the cursor | Fixed to aim from the player's current position |
| Pickup frequency / placement | Centre-clustered; map felt empty elsewhere | Random spawning across the whole map, maintained at a steady count |
| UI clarity | Bare numbers were unreadable mid-combat | Re-designed labelled, colour-coded HUD |
| Audio feedback | Shoot/pickup sounds reinforced actions well | Kept; background music added |
| Game-over moment | Clear cause (health hits 0) and a dedicated Game Over screen | Added menu / pause / game-over screens for a complete flow |

---

## Part 2: Implementation Document

All gameplay logic is implemented with **Unity Visual Scripting** (Script Graph assets) — there is no hand-written C# gameplay code. Logic communicates through **Scene Variables** and **Custom Events**.

### Feature 1 – WebGL Build

| | |
|---|---|
| **Build target** | WebGL (Unity 6000.3.7f1) |
| **Compression** | Brotli with **Decompression Fallback** enabled (loads on itch.io without custom server headers) |
| **Output** | `Build/WebGL/` (`index.html` + `Build/` + `TemplateData/`), packaged as `DesertSurvivor_WebGL.zip` |
| **Hosted at** | itch.io (HTML project, "played in browser", 1280×720 viewport) |

The build was produced via `BuildPipeline.BuildPlayer` targeting WebGL. Decompression Fallback was enabled in *Player Settings → Publishing Settings* so the compressed `.unityweb` files decompress in-browser, which is required for itch.io hosting.

---

### Feature 2 – Player Avatar

| | |
|---|---|
| **GameObject** | `Player` (Tag: Player, Layer 6) |
| **Script Machines** | `MovementScript`, `PlayerShoot`, `ReloadLogic` |
| **Sprite** | Kenney Roguelike Characters 2 |
| **Physics** | Rigidbody2D (Dynamic, GravityScale = 0, FreezeRotationZ) |
| **Collider** | CapsuleCollider2D |

**Unity features used:**
- `Rigidbody2D.linearVelocity` set each frame for physics-based, frame-rate-independent movement.
- New Input System (`COMP1151.inputactions`) — Player/Move action (Vector2) bound to WASD / arrow keys.
- Visual Script: `On Update` → read movement input → set `Rigidbody2D.linearVelocity = direction × speed`.
- FreezeRotationZ prevents collisions from spinning the sprite.

---

### Feature 3 – World & Camera

| | |
|---|---|
| **Tilemaps** | `Grid` → `Ground` (varied desert sand), `Buildings` (solid), `Decor` (props) |
| **Camera** | `CinemachineBrain` on Main Camera; `CM camera` (CinemachineCamera + CinemachineFollow) tracks the Player |
| **Camera background** | Solid desert tone; Orthographic, size ≈ 10–11 |

**Unity features used:**
- Cinemachine: `CinemachineCamera` with `CinemachineFollow` targeting the Player's Transform; `CinemachineBrain` drives the Main Camera.
- Three `Tilemap` layers on a single `Grid`, sorted so the player/hazards render above the world.
- The map was painted programmatically from the Kenney tile assets into a desert town with buildings, scattered cacti/rocks/crates and varied ground.

---

### Feature 4 – Obstacles & Boundaries

| | |
|---|---|
| **Buildings** | `Buildings` Tilemap — `Tilemap Collider 2D` + `Composite Collider 2D` + static `Rigidbody2D` |
| **Boundary** | `Walls` GameObject — four `BoxCollider2D` edges enclosing the map |
| **Physics Layer** | Default (Layer 0), which collides with Player (Layer 6) in the 2D matrix |

**Unity features used:**
- `Tilemap Collider 2D` (Used by Composite) + `Composite Collider 2D` merge the building tiles into efficient solid geometry the player must navigate around.
- The four boundary box colliders keep the player inside the play space.
- The Player↔Default collision was verified against the *Project Settings → Physics 2D* layer matrix.

---

### Feature 5 – Pickups

| | |
|---|---|
| **Prefab** | `Assets/Resources/Pickup.prefab` (Tag: Pickup) |
| **Script Machine** | `PickupLogic` (on the prefab) |
| **Spawner** | `PickupSpawner` GameObject — Script Machine `PickupSpawner` |
| **Sprite** | Kenney Desert Shooter Pack |

**Unity features used:**
- Trigger `CircleCollider2D` (IsTrigger). `PickupLogic`: `On Trigger Enter 2D` → `CompareTag("Player")` → `score + 1` → `Trigger Custom Event "ScoreChanged"` → play coin SFX → `Destroy(self)`.
- `PickupSpawner` (gated to only run while playing): counts live pickups with `GameObject.FindGameObjectsWithTag("Pickup")`; while below the target count it `Instantiate`s `Resources.Load("Pickup")` at a random map position (`Random.Range`), so coins continuously appear across the whole map.

---

### Feature 6 – Hazards & Wave System

| | |
|---|---|
| **Prefab** | `Assets/Resources/Hazard.prefab` (Tag: Hazard, Layer 9, Object Variable `hp`) |
| **Script Machine** | `HazardMovement` (on the prefab) |
| **Wave manager** | `WaveManager` GameObject — Script Machine `WaveManager` |
| **Sprite** | Kenney Roguelike Characters 2 |

**Unity features used:**
- `HazardMovement`: on `FixedUpdate`, finds the Player (`FindWithTag`) and sets `linearVelocity` toward it (chase, speed 2.5). `On Trigger Enter 2D` with `CompareTag("Player")` decreases `health` (guarded by `health > 0`) and fires `Trigger Custom Event "HealthChanged"`. On `Start` it sets its own `hp` Object Variable equal to the current `wave`, so later waves are tougher.
- Two colliders per hazard: a trigger (damage) and a solid `CircleCollider2D` with a per-collider **Layer Override** so hazards push apart from each other (Layer 9 ↔ 9) but not the player.
- `WaveManager` (gated to only run while playing): spawns a batch of hazards one-per-frame at **player-relative** positions (`playerPos ± Random.Range(...)` clamped to the map with `Mathf.Clamp`); when the batch is placed and `FindGameObjectsWithTag("Hazard").Length == 0`, it increments `wave` and queues the next, larger batch (`count = wave + 2`).

---

### Feature 7 – Shooting (Gun, Bullets, Reload)

| | |
|---|---|
| **Gun pickup** | `GunPickup` prefab → `GunPickupLogic` (sets `hasGun = true`, destroys self) |
| **Firing** | `PlayerShoot` Script Machine on the Player |
| **Reload** | `ReloadLogic` Script Machine on the Player |
| **Bullet** | `Assets/Resources/Bullet.prefab` → `BulletLogic` |

**Unity features used:**
- `PlayerShoot` (gated to only run while playing): on left mouse click, if `hasGun && !isReloading && ammo > 0`, `Instantiate`s `Resources.Load("Bullet")` at the player, decrements `ammo`, and plays the shoot SFX.
- `BulletLogic`: on `Start`, aims from the **player's current position** to the mouse world point (via `Camera.main.ScreenToWorldPoint`, z zeroed for 2D) and sets velocity at speed 10; auto-destroys after 3 s. On hitting a "Hazard" it subtracts from that hazard's `hp` Object Variable and destroys it only when `hp ≤ 0`.
- `ReloadLogic`: pressing **R** starts a 1.5 s timer (`reloadTimer`), blocking firing until it completes and restores `ammo` to 6.

---

### Feature 8 – UI / HUD

| | |
|---|---|
| **Canvas** | `Canvas` (Screen Space – Overlay) |
| **HUD** | `HUD` panel (top-left) with labelled values: `WaveText`, `HealthText`, `ScoreText`, `AmmoText` |
| **Script Machine** | `UIManager` |

**Unity features used:**
- `TextMeshProUGUI` for all readouts, arranged in a translucent panel with colour-coded labels (WAVE / HEALTH / SCORE / AMMO).
- `UIManager` (`On Update`): finds each text object by name, reads the matching Scene Variable, and writes it into the `.text` field every frame.
- RectTransform anchors keep the HUD pinned to the corner at any resolution.

---

### Feature 9 – Game State & Screens

| | |
|---|---|
| **Manager** | `GameManager` GameObject — Script Machine `GameManager` |
| **State var** | `gameState` (0 = loading, 1 = menu, 2 = playing, 3 = paused, 4 = game over) |
| **Screens** | `LoadingPanel`, `MenuPanel`, `PausePanel`, `GameOverPanel` (children of the Canvas) |
| **Buttons** | `PlayBtn`, `QuitBtn`, `ResumeBtn`, `RestartBtn`, `PlayAgainBtn` |

**Unity features used:**
- `GameManager` (`On Update`): syncs `Time.timeScale` (1 while playing, else 0); advances loading → menu after 1.5 s of `Time.unscaledTime`; toggles pause with the **Esc** key; sets `gameState = 4` when `health == 0`; and shows/hides each panel by comparing `gameState` and calling `SetActive` (panels are located with `Transform.Find` so inactive ones can still be reached).
- Buttons use Visual Scripting's **`On Button Click`** event: Play/Resume set `gameState = 2`; Restart/Play Again call `SceneManager.LoadScene(0)` for a clean reset; Quit calls `Application.Quit`.
- Gameplay graphs (`WaveManager`, `PlayerShoot`) are gated behind `gameState == 2`, so nothing spawns or fires on the menu, loading, or pause screens.

---

### Feature 10 – Audio

| | |
|---|---|
| **BGM** | `AudioSource` on Main Camera — loop, Play On Awake, low volume (placeholder Kenney track) |
| **Shoot SFX** | `AudioSource` on Player (`shoot-a`), played from `PlayerShoot` |
| **Coin SFX** | `CoinSFX` GameObject `AudioSource` (`coin-a`), played from `PickupLogic` |

**Unity features used:**
- `AudioSource.Play()` invoked from Visual Scripting at the firing and pickup events.
- The coin sound lives on a persistent `CoinSFX` object so it still plays after the collected pickup is destroyed.
- The Main Camera carries a looping background-music `AudioSource` (currently a placeholder clip from the Kenney pack; intended to be swapped for a dedicated music track).

---

### Scene Variables

| Name | Type | Initial |
|---|---|---|
| `health` | Integer | 5 |
| `score` | Integer | 0 |
| `hasGun` | Boolean | false |
| `ammo` | Integer | 6 |
| `isReloading` | Boolean | false |
| `reloadTimer` | Float | 0 |
| `wave` | Integer | 0 |
| `toSpawn` | Integer | 0 |
| `gameState` | Integer | 0 |

### Physics Layers

`Player` = Layer 6, `Hazard` = Layer 9, walls/buildings = Default (0). The 2D layer matrix enables Hazard↔Player (damage), Hazard↔Hazard (separation) and Player↔Default (walls).

---

## Credits / Third-Party Assets

| Asset | Author | URL | License | Used for |
|---|---|---|---|---|
| Desert Shooter Pack | Kenney | https://kenney.nl/assets/desert-shooter-pack | CC0 1.0 | Tiles, gun/bullet & item sprites, all sound effects, placeholder music |
| Roguelike Characters 2 | Kenney | https://kenney.nl/assets/roguelike-characters-2 | CC0 1.0 | Player and hazard character sprites |
| Cinemachine | Unity Technologies | https://unity.com/features/cinemachine | Unity Companion License | Follow camera |
| TextMesh Pro | Unity Technologies | (built-in package) | Unity Companion License | UI text |
| Visual Scripting | Unity Technologies | (built-in package) | Unity Companion License | All gameplay logic |
| Input System | Unity Technologies | (built-in package) | Unity Companion License | Player movement input |

*Note: the background music is currently a placeholder sound effect from the Kenney Desert Shooter Pack (CC0). If a dedicated music track is added before submission, credit it here.*
