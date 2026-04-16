# DinazorKac -- Tasks

> Ordered work breakdown derived from IMPLEMENTATION.md.
> Execute sequentially. Each task is completable in a single session.

## Summary

| Metric              | Value                |
|---------------------|----------------------|
| Total Tasks         | 20                   |
| Phases              | 6                    |
| Foundation Complete | After Task 3         |
| Core Gameplay MVP   | After Task 10        |
| Full Release        | After Task 20        |

---

## Phase 1: Project Foundation

> Establishes project structure, core types, build system, and configuration.
> After this phase: project compiles, dev server runs, empty Phaser canvas renders.

### Task 1: Project Scaffolding

**Create the project skeleton with all config files, dependencies, and directory structure.**

> **Working directory:** The project root already exists and is the current working
> directory. **Do NOT create or `cd` into a subfolder.** Every file path below is
> relative to CWD. Leave the planning documents (SPECIFICATION.md, IMPLEMENTATION.md,
> TASKS.md, BRANDING.md, PROMPT.md) untouched.

**Files to create:**
- `package.json` -- project metadata, all dependencies from IMPLEMENTATION.md §1.3, npm scripts
- `tsconfig.json` -- TypeScript strict mode, ES2020 target, DOM lib
- `vite.config.ts` -- Vite configuration from IMPLEMENTATION.md §11.2
- `.eslintrc.json` -- ESLint flat config with @typescript-eslint
- `.prettierrc` -- 2 space indent, single quotes, trailing comma
- `.gitignore` -- node_modules, dist, .env, coverage, .DS_Store
- `index.html` -- minimal HTML shell with `<div id="game-container">`
- `LICENSE` -- MIT license
- `README.md` -- minimal: project name, description, setup instructions

**Directories to create (empty with `.gitkeep`):**
- `src/config/`
- `src/scenes/`
- `src/entities/`
- `src/managers/`
- `src/ui/`
- `src/events/`
- `src/types/`
- `assets/sprites/`
- `assets/audio/music/`
- `assets/audio/sfx/`
- `assets/fonts/`
- `public/`
- `tests/managers/`
- `tests/config/`

**Commands to run:**
```bash
npm init -y
npm install phaser@4
npm install -D typescript vite vitest eslint prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

**Acceptance Criteria:**
- [ ] `npx tsc --noEmit` completes without errors
- [ ] `npm run dev` starts Vite dev server
- [ ] `npm run lint` passes
- [ ] All directories from IMPLEMENTATION.md §3.1 exist
- [ ] `.gitignore` covers node_modules, dist, coverage, .env

**Dependencies:** None
**Refs:** IMPLEMENTATION.md §1, §3.1, §11.2, §12.1

---

### Task 2: Core Types & Constants

**Define all shared TypeScript types, game events, and configuration constants.**

**Files to create:**
- `src/types/index.ts` -- All shared types: `GameState`, `DifficultyLevel`, `MeteorSize`, `PowerUpType`, `HighScoreData`, `GameSettings`, `StorageData`, `DifficultyParams`
- `src/events/GameEvents.ts` -- Event name constants from IMPLEMENTATION.md §2.3
- `src/config/AssetKeys.ts` -- Asset key string constants for sprites, audio, fonts
- `src/config/GameConfig.ts` -- Phaser game configuration object from IMPLEMENTATION.md §6.2
- `src/config/DifficultyConfig.ts` -- Difficulty presets table from IMPLEMENTATION.md §2.5

**Code requirements:**
- All types from SPECIFICATION.md §5.1 (GameState, Player, Meteor, PowerUp entities)
- StorageData interface matching SPECIFICATION.md §5.2 localStorage schema
- DifficultyParams interface with all parameters from SPECIFICATION.md §3.4.1 table
- Game constants from IMPLEMENTATION.md §8.1 (GAME_WIDTH, GAME_HEIGHT, METEOR_MAX_COUNT, etc.)
- No `any` types anywhere

**Acceptance Criteria:**
- [ ] `npx tsc --noEmit` passes with zero errors
- [ ] All entity types from SPECIFICATION.md §5.1 exist as TypeScript interfaces
- [ ] `difficultyPresets` object has `easy`, `medium`, `hard` entries matching SPEC §3.4.1 table values
- [ ] `GameEvents` object has all events from IMPLEMENTATION.md §2.3
- [ ] `AssetKeys` has organized string constants for sprites, audio, fonts

**Dependencies:** Task 1
**Refs:** SPECIFICATION.md §2, §5; IMPLEMENTATION.md §2.3, §2.5, §6.2, §8.1

---

### Task 3: Entry Point & Boot Scene

**Create the application entry point and BootScene that loads all game assets.**

**Files to create:**
- `src/main.ts` -- Application entry point: create `new Phaser.Game(gameConfig)`, import all scenes
- `src/scenes/BootScene.ts` -- Asset loading scene with progress bar

**Implementation:**
1. `main.ts`: Import GameConfig, instantiate Phaser.Game
2. `BootScene.create()`: Display loading progress bar (simple rectangle graphics)
3. `BootScene.preload()`: Load all sprite sheets, audio files, bitmap fonts using AssetKeys
4. On load complete: transition to MenuScene
5. Use placeholder colored rectangles for sprites (actual pixel art comes in Phase 4)
6. Use Phaser.GameObjects.Graphics for the progress bar

**Acceptance Criteria:**
- [ ] `npm run dev` shows a loading screen, then transitions to an empty MenuScene
- [ ] Browser console has no errors
- [ ] Phaser canvas renders at 480x720 design size with FIT scaling
- [ ] Canvas auto-centers in the browser window
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 2
**Refs:** SPECIFICATION.md §7.2 (Boot Screen); IMPLEMENTATION.md §6.1, §6.2

---

## Phase 2: Core Entities & Managers

> Implements all game entity classes and management systems.
> After this phase: all building blocks exist, not yet connected in a scene.

### Task 4: StorageManager

**Implement localStorage wrapper for high scores and settings persistence.**

**Files to create:**
- `src/managers/StorageManager.ts` -- Full implementation from IMPLEMENTATION.md §4.3
- `tests/managers/StorageManager.test.ts` -- Unit tests

**Implementation:**
1. Static class with `load()`, `save()`, `updateHighScore()`, `updateSettings()`, `getDefaults()`
2. JSON serialization/deserialization with try-catch for localStorage access
3. Fallback to defaults when localStorage unavailable or corrupted
4. Use `StorageData` type from `src/types/index.ts`

**Test requirements:**
- Load returns defaults when localStorage is empty
- Load returns saved data when present
- Load returns defaults when data is corrupted JSON
- updateHighScore returns true and saves when new score is higher
- updateHighScore returns false and doesn't save when score is lower
- save handles localStorage quota error gracefully

**Acceptance Criteria:**
- [ ] `npm run test` passes all StorageManager tests
- [ ] localStorage operations wrapped in try-catch
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 2
**Refs:** SPECIFICATION.md §3.5.1, §5.2, §5.3; IMPLEMENTATION.md §4.2, §4.3

---

### Task 5: InputManager

**Implement unified input handler for keyboard, mouse, and touch controls.**

**Files to create:**
- `src/managers/InputManager.ts` -- Keyboard (arrows + WASD), mouse, and touch input

**Implementation:**
1. `setup(scene)` method initializes cursor keys, WASD keys, and pointer
2. `getMovementX(playerX)` returns -1 (left), 0 (none), or 1 (right)
3. Keyboard input takes priority over pointer
4. Pointer: track X position relative to player, apply dead zone of 10px
5. Follow pattern from IMPLEMENTATION.md §6.3

**Acceptance Criteria:**
- [ ] Left/Right arrow keys return correct movement direction
- [ ] A/D keys return correct movement direction
- [ ] Mouse/touch pointer isDown tracks horizontal diff from player
- [ ] Dead zone prevents jitter when pointer is near player
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 2
**Refs:** SPECIFICATION.md §3.1.1; IMPLEMENTATION.md §6.3

---

### Task 6: Player Entity

**Implement the dinosaur player sprite with movement and shield state.**

**Files to create:**
- `src/entities/Player.ts` -- Player class extending Phaser.Physics.Arcade.Sprite

**Implementation:**
1. Constructor: set position at bottom of screen (GAME_HEIGHT - PLAYER_Y_OFFSET), set hitbox to 80% of sprite
2. `update(delta, movementX)`: apply horizontal movement based on speed, clamp within screen bounds
3. `activateShield()`: set hasShield flag, show shield visual (tinted overlay or separate sprite)
4. `deactivateShield()`: remove shield visual, play break animation
5. `hasShieldActive()`: getter for shield state
6. Use placeholder rectangle graphic until pixel art is added

**Acceptance Criteria:**
- [ ] Player renders at bottom center of screen
- [ ] Player moves left/right within screen bounds
- [ ] Player cannot move beyond screen edges
- [ ] Shield visual appears/disappears correctly
- [ ] Hitbox is 80% of visual sprite size
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 2
**Refs:** SPECIFICATION.md §3.1.1, §3.3.2; IMPLEMENTATION.md §3.2 (Entities module)

---

### Task 7: Meteor Entity & Spawner

**Implement the Meteor entity and MeteorSpawner with object pooling.**

**Files to create:**
- `src/entities/Meteor.ts` -- Meteor class extending Phaser.Physics.Arcade.Sprite
- `src/managers/MeteorSpawner.ts` -- Meteor pool manager with spawn logic

**Implementation:**
1. `Meteor` class: three sizes (small: 24px, medium: 40px, large: 56px), rotation animation, activate/deactivate for pool
2. `Meteor.activate(speed, size)`: reset position to top of screen, set velocity, make visible
3. `Meteor.preUpdate()`: if y > GAME_HEIGHT + 50, deactivate and return to pool
4. `MeteorSpawner`: use Phaser.Physics.Arcade.Group with maxSize: 30 (SPEC §3.1.2)
5. `MeteorSpawner.spawn()`: get from pool, set random X, random size, current speed
6. Random X distribution: ensure minimum spread to avoid unplayable clustering
7. Use placeholder colored circles until pixel art is added

**Acceptance Criteria:**
- [ ] Meteors spawn at top with random X positions
- [ ] Three meteor sizes render with correct dimensions
- [ ] Meteors fall at configured speed
- [ ] Meteors deactivate when passing below screen
- [ ] Pool maxSize of 30 is enforced
- [ ] Rotation animation plays during fall
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 2
**Refs:** SPECIFICATION.md §3.1.2; IMPLEMENTATION.md §2.4, §3.2

---

### Task 8: PowerUp Entity & Factory

**Implement the PowerUp entity, factory, and PowerUpManager.**

**Files to create:**
- `src/entities/PowerUp.ts` -- PowerUp class extending Phaser.Physics.Arcade.Sprite
- `src/managers/PowerUpManager.ts` -- Power-up spawning, active effects tracking, expiration

**Implementation:**
1. `PowerUp` class: three types (shield, slowMotion, doublePoints), distinct visual per type
2. PowerUp falls slower than meteors (POWER_UP_FALL_SPEED: 100px/s)
3. `PowerUpManager`:
   - Tracks active effects with remaining duration
   - `trySpawn()`: check timer against difficulty's powerUpInterval, spawn random type at random X
   - `activate(type)`: apply effect, emit POWER_UP_COLLECTED event
   - `update(delta)`: decrease active durations, emit POWER_UP_EXPIRED when timer ends
   - slowMotion expiry: gradual speed restore over 1 second (not instant)
   - Stacking: reset timer, don't multiply effect (SPEC §3.3.3, §3.3.4)
4. Use placeholder colored shapes (blue circle=shield, green diamond=slow, gold star=doublePoints)

**Acceptance Criteria:**
- [ ] Three power-up types render with distinct visuals
- [ ] Power-ups fall at configured speed (slower than meteors)
- [ ] Shield activates on player and persists until hit
- [ ] Slow motion reduces meteor speed by 50% for 5 seconds
- [ ] Double points doubles score rate for 8 seconds
- [ ] Stacking resets timer instead of multiplying
- [ ] Events emitted on collect and expire
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 6, Task 7
**Refs:** SPECIFICATION.md §3.3; IMPLEMENTATION.md §2.6

---

### Task 9: ScoreManager & DifficultyManager

**Implement score calculation with multipliers and progressive difficulty scaling.**

**Files to create:**
- `src/managers/ScoreManager.ts` -- Score tracking, double points multiplier, high score comparison
- `src/managers/DifficultyManager.ts` -- Progressive difficulty scaling over time
- `tests/managers/ScoreManager.test.ts` -- Score calculation tests
- `tests/managers/DifficultyManager.test.ts` -- Difficulty scaling tests

**Implementation:**
1. `ScoreManager`:
   - `update(delta)`: add SCORE_PER_TICK per 100ms, apply x2 if doublePoints active
   - `activateDoublePoints()` / `deactivateDoublePoints()`
   - `getScore()`: return current score
   - `checkHighScore(difficulty)`: compare with StorageManager, return boolean
   - Format score with thousands separator for display
2. `DifficultyManager`:
   - Constructor takes DifficultyParams from selected preset
   - `update(delta)`: every DIFFICULTY_TICK_INTERVAL, increase speed and decrease spawn rate
   - `getCurrentSpeed()`: return current meteor speed (capped at MAX_METEOR_SPEED)
   - `getCurrentSpawnRate()`: return current spawn interval (minimum floor)
   - Speed increase formula: `speed *= (1 + speedIncreaseRate)` per tick

**Test requirements:**
- ScoreManager: score increments correctly per tick, double points doubles rate, format includes commas
- DifficultyManager: speed increases correctly per interval, respects maximum cap, different presets

**Acceptance Criteria:**
- [ ] Score increments by 1 per 100ms
- [ ] Double points doubles the increment rate
- [ ] Score formatted with thousands separator: "1,234"
- [ ] Difficulty increases every 10 seconds
- [ ] Meteor speed capped at MAX_METEOR_SPEED (600px/s)
- [ ] All tests pass
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 4
**Refs:** SPECIFICATION.md §3.1.4, §3.4.2; IMPLEMENTATION.md §2.5, §3.2

---

### Task 10: AudioManager

**Implement centralized audio control for music and sound effects.**

**Files to create:**
- `src/managers/AudioManager.ts` -- Music playback, SFX playback, volume/mute controls

**Implementation:**
1. Singleton pattern scoped to Phaser.Scene (passed via scene reference)
2. `playMusic(key)`: play background music in loop, handle browser autoplay policy
3. `stopMusic()`: stop current music
4. `playSfx(key)`: play one-shot sound effect
5. `setMusicEnabled(enabled)` / `setSfxEnabled(enabled)`: toggle and persist to StorageManager
6. `isMusicEnabled()` / `isSfxEnabled()`: getters
7. Handle browser autoplay restriction: defer music start until first user interaction
8. Load settings from StorageManager on init

**Acceptance Criteria:**
- [ ] Music plays in loop when enabled
- [ ] SFX plays on demand when enabled
- [ ] Music/SFX can be toggled independently
- [ ] Settings persist to localStorage via StorageManager
- [ ] Browser autoplay policy handled (no console errors)
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 4
**Refs:** SPECIFICATION.md §3.6; IMPLEMENTATION.md §3.2

---

## Phase 3: Game Scenes & Integration

> Connects all entities and managers into functional game scenes.
> After this phase: complete playable game loop with placeholder graphics.

### Task 11: MenuScene

**Implement the main menu screen with title, play button, difficulty selector, and settings.**

**Files to create:**
- `src/scenes/MenuScene.ts` -- Full menu scene
- `src/ui/Button.ts` -- Reusable interactive button class
- `src/ui/DifficultySelector.ts` -- Three-option difficulty selector

**Implementation:**
1. Display "DinazorKac" title text (Phaser.GameObjects.Text, pixel font style)
2. "Oyna" button -- navigates to GameScene with selected difficulty
3. DifficultySelector: three toggle options (Kolay/Orta/Zor), highlight selected, default from StorageManager
4. High score display for selected difficulty (updates when difficulty changes)
5. Music/SFX toggle buttons (speaker icons or text toggles)
6. Start menu music via AudioManager
7. Button class: interactive zone with hover/press states, click callback, touch support
8. All interactive elements must be minimum 44x44px for mobile

**Acceptance Criteria:**
- [ ] Title "DinazorKac" displays prominently
- [ ] "Oyna" button starts the game with correct difficulty
- [ ] Difficulty selector shows three options with visual highlight on selected
- [ ] High score updates when difficulty selection changes
- [ ] Music/SFX toggles work and persist
- [ ] All buttons/touch targets are minimum 44x44px
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 3, Task 4, Task 10
**Refs:** SPECIFICATION.md §3.2.1, §7.2; IMPLEMENTATION.md §6.1

---

### Task 12: GameScene -- Core Loop

**Implement the main game scene: player movement, meteor spawning, collision, and scoring.**

**Files to create/modify:**
- `src/scenes/GameScene.ts` -- Main game scene with full game loop
- `src/ui/HUD.ts` -- In-game heads-up display

**Implementation:**
1. `create()`:
   - Initialize Player at bottom center
   - Initialize MeteorSpawner, PowerUpManager, ScoreManager, DifficultyManager, InputManager, AudioManager
   - Set up collision detection: player vs meteors, player vs power-ups
   - Create HUD
   - Start game music
2. `update(time, delta)`:
   - Get movement from InputManager, pass to Player
   - Update MeteorSpawner: check spawn timer, spawn if due
   - Update PowerUpManager: check spawn timer, update active effects
   - Update ScoreManager: increment score
   - Update DifficultyManager: check for speed/rate increase
   - Apply DifficultyManager values to MeteorSpawner
   - Update HUD with current score, active power-ups
3. Collision: meteor-player:
   - If shield active: break shield, destroy meteor, play shield-break SFX
   - If no shield: trigger Game Over -- emit GAME_OVER event, transition to GameOverScene
   - Visual feedback: screen flash or player tint
4. Collision: powerup-player:
   - Activate power-up effect via PowerUpManager
   - Play collect SFX
   - Destroy power-up sprite
5. HUD:
   - Score display (right top corner)
   - Active power-up icons with remaining time (left top corner)
   - Pause button (right edge, mobile-friendly size)

**Acceptance Criteria:**
- [ ] Player moves with keyboard, mouse, and touch
- [ ] Meteors spawn from top at configured rate and speed
- [ ] Difficulty increases over time (visible speed change)
- [ ] Power-ups spawn at configured intervals
- [ ] Shield absorbs one hit, then breaks
- [ ] Slow motion visibly slows all meteors
- [ ] Double points shows x2 indicator on score
- [ ] Meteor collision without shield triggers game over
- [ ] Score increments continuously and displays formatted
- [ ] HUD shows score and active power-ups
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 5, Task 6, Task 7, Task 8, Task 9, Task 10
**Refs:** SPECIFICATION.md §3.1, §3.2.2, §3.3; IMPLEMENTATION.md §2.1, §2.2, §6.1

---

### Task 13: Pause System

**Implement pause functionality with overlay UI.**

**Files to create:**
- `src/ui/PauseOverlay.ts` -- Pause menu overlay

**Files to modify:**
- `src/scenes/GameScene.ts` -- Add pause/resume logic
- `src/managers/InputManager.ts` -- Add Escape key binding for pause

**Implementation:**
1. Escape key or pause button triggers pause
2. `GameScene` pause: stop physics, stop timers, stop score updates
3. PauseOverlay: semi-transparent dark background overlay
4. Three buttons: "Devam Et" (resume), "Yeniden Basla" (restart GameScene), "Ana Menu" (go to MenuScene)
5. Resume: remove overlay, restart physics/timers
6. Mobile: pause button in HUD (not triggered by random screen taps)

**Acceptance Criteria:**
- [ ] Escape key pauses the game
- [ ] Pause button in HUD pauses the game
- [ ] All game logic stops during pause (meteors freeze, score stops)
- [ ] "Devam Et" resumes gameplay from exact state
- [ ] "Yeniden Basla" starts a fresh game with same difficulty
- [ ] "Ana Menu" returns to MenuScene
- [ ] Random touch on game area does NOT trigger pause
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 12
**Refs:** SPECIFICATION.md §3.2.4; IMPLEMENTATION.md §2.2

---

### Task 14: GameOverScene

**Implement the game over screen with score display, high score check, and replay options.**

**Files to create:**
- `src/scenes/GameOverScene.ts` -- Game over screen

**Implementation:**
1. Receive final score and difficulty from GameScene (via scene data)
2. Display "Game Over" text with entrance animation (scale + fade)
3. Show final score and high score for current difficulty
4. Check if new high score via StorageManager.updateHighScore()
5. If new record: show "Yeni Rekor!" with special animation (gold text, particles or flash)
6. Play new-record SFX or game-over SFX accordingly
7. "Tekrar Oyna" button: restart GameScene with same difficulty
8. "Ana Menu" button: go to MenuScene
9. Quick restart: any key press or tap (after short delay to prevent accidental restart)

**Acceptance Criteria:**
- [ ] "Game Over" text animates in
- [ ] Final score displays correctly
- [ ] High score displays for current difficulty
- [ ] New record detection works and shows "Yeni Rekor!" animation
- [ ] Correct SFX plays (game over vs new record)
- [ ] "Tekrar Oyna" restarts game with same difficulty
- [ ] "Ana Menu" returns to menu
- [ ] Quick restart works after 1-second delay
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 12
**Refs:** SPECIFICATION.md §3.2.3, §3.5.1; IMPLEMENTATION.md §6.1

---

## Phase 4: Visual Assets & Audio

> Creates and integrates pixel art assets and audio files.
> After this phase: game looks and sounds complete.

### Task 15: Pixel Art Sprites

**Create all pixel art sprites and integrate into the game.**

**Files to create:**
- `assets/sprites/dino-sheet.png` -- Dinosaur sprite sheet (idle, walk-left, walk-right, shield)
- `assets/sprites/meteors.png` -- Meteor variations (small, medium, large, 2-3 variants each)
- `assets/sprites/powerups.png` -- Power-up icons (shield=blue, slowMotion=green, doublePoints=gold)
- `assets/sprites/background.png` -- Scrolling background or static backdrop
- `assets/sprites/ui-atlas.png` -- UI elements (buttons, pause icon, speaker icon)

**Files to modify:**
- `src/scenes/BootScene.ts` -- Update asset loading to use actual sprite sheets
- `src/entities/Player.ts` -- Replace placeholder with sprite sheet animations
- `src/entities/Meteor.ts` -- Replace placeholder with meteor sprites
- `src/entities/PowerUp.ts` -- Replace placeholder with power-up sprites
- `src/scenes/GameScene.ts` -- Add background sprite
- `src/ui/Button.ts` -- Use UI atlas for button graphics

**Implementation:**
1. Create pixel art assets at 2x resolution for crisp scaling
2. Dinosaur: 4-frame walk cycle, idle pose, shield overlay frame
3. Meteors: rocky/fiery appearance, 2-3 variants per size for variety
4. Power-ups: clearly distinct colors and shapes (intuitive at a glance)
5. Background: dark sky/space theme with stars, subtle color gradient
6. Use Phaser Sprite.anims for dinosaur movement animation
7. Particle effects for meteor trails (Phaser Particles)

**Acceptance Criteria:**
- [ ] Dinosaur has walk animation when moving
- [ ] Dinosaur has idle pose when stationary
- [ ] Shield visual renders around dinosaur when active
- [ ] Three meteor sizes render with distinct sprites
- [ ] Three power-up types are visually distinguishable
- [ ] Background renders behind gameplay elements
- [ ] UI buttons use atlas sprites
- [ ] Total asset size < 2MB
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 14
**Refs:** SPECIFICATION.md §1.3 (renkli pixel art); IMPLEMENTATION.md §1.2 (asset format)

---

### Task 16: Audio Integration

**Create or source audio files and integrate with AudioManager.**

**Files to create:**
- `assets/audio/music/menu-theme.mp3` -- Menu background music (looping, calm retro)
- `assets/audio/music/game-theme.mp3` -- Game background music (looping, upbeat retro)
- `assets/audio/sfx/hit.mp3` -- Meteor collision sound
- `assets/audio/sfx/powerup-collect.mp3` -- Power-up pickup sound
- `assets/audio/sfx/shield-break.mp3` -- Shield breaking sound
- `assets/audio/sfx/gameover.mp3` -- Game over sound
- `assets/audio/sfx/new-record.mp3` -- New high score celebration
- `assets/audio/sfx/button-click.mp3` -- UI button click

**Files to modify:**
- `src/scenes/BootScene.ts` -- Load all audio assets
- `src/config/AssetKeys.ts` -- Ensure all audio keys match file names
- `src/scenes/MenuScene.ts` -- Play menu music, button click SFX
- `src/scenes/GameScene.ts` -- Play game music, all gameplay SFX
- `src/scenes/GameOverScene.ts` -- Play game over / new record SFX

**Implementation:**
1. Source royalty-free audio or generate with tools (jsfxr for SFX, royalty-free loops for music)
2. MP3 format for broad compatibility
3. Music: normalize volume, ensure seamless loop points
4. SFX: short clips (< 2 seconds each), normalized volume
5. Wire all SFX to correct game events via AudioManager
6. Handle browser autoplay: resume AudioContext on first user interaction

**Acceptance Criteria:**
- [ ] Menu music plays on MenuScene entry
- [ ] Game music plays during gameplay (different from menu)
- [ ] All 6 SFX play at correct moments
- [ ] Music loops seamlessly
- [ ] Music/SFX toggles work correctly
- [ ] No audio console errors on any browser
- [ ] Browser autoplay policy handled
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 14
**Refs:** SPECIFICATION.md §3.6; IMPLEMENTATION.md §3.2

---

## Phase 5: Polish & Responsive

> Visual polish, animations, responsive behavior, and mobile optimization.
> After this phase: game is polished and plays well on all devices.

### Task 17: Visual Effects & Animations

**Add particle effects, screen effects, and UI animations for polish.**

**Files to create/modify:**
- `src/scenes/GameScene.ts` -- Screen flash on hit, meteor trail particles
- `src/scenes/GameOverScene.ts` -- Score counter animation, new record particles
- `src/entities/Meteor.ts` -- Rotation animation, trail particles
- `src/entities/Player.ts` -- Hit reaction animation (tint flash)
- `src/ui/HUD.ts` -- Score pop animation on milestone, power-up timer bar animation

**Implementation:**
1. Meteor trail: subtle particle emitter following each meteor (fire/smoke particles)
2. Screen flash: brief white overlay on collision (100ms fade)
3. Player hit: red tint flash on damage (shield break or game over)
4. Score milestone: brief scale-up animation at 100, 500, 1000+ intervals
5. New record: gold particle burst on GameOverScene
6. Power-up timer: smooth shrinking progress bar under active power-up icons
7. Slow motion visual: slight blue color tint or desaturation on game area
8. Menu transitions: scene fade-in/fade-out between scenes

**Acceptance Criteria:**
- [ ] Meteors have visible trail particles
- [ ] Screen flash occurs on collision
- [ ] Player tint flash on damage/death
- [ ] Score animates at milestones
- [ ] New record has celebratory particle effect
- [ ] Power-up timers visually deplete
- [ ] Slow motion has visual indicator
- [ ] Scene transitions are smooth (no harsh cuts)
- [ ] All effects maintain 60 FPS
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 15, Task 16
**Refs:** SPECIFICATION.md §3.1.3, §3.2.3, §3.3

---

### Task 18: Mobile Optimization & Responsive

**Optimize touch controls, responsive scaling, and mobile UX.**

**Files to modify:**
- `src/config/GameConfig.ts` -- Verify Scale.FIT and CENTER_BOTH settings
- `src/managers/InputManager.ts` -- Polish touch/drag controls, add touch dead zone
- `src/ui/HUD.ts` -- Ensure all UI elements are readable on mobile
- `src/ui/Button.ts` -- Verify 44x44px minimum touch targets
- `src/ui/PauseOverlay.ts` -- Mobile-friendly button sizes
- `src/scenes/MenuScene.ts` -- Mobile layout adjustments
- `src/scenes/GameOverScene.ts` -- Mobile layout adjustments

**Implementation:**
1. Touch controls: smooth drag-to-move (player follows finger X position)
2. Prevent default touch behaviors (scroll, zoom) on game canvas
3. Scale all UI text relative to canvas size
4. Test at breakpoints: 320px, 375px, 414px (phones), 768px (tablet), 1024px+ (desktop)
5. Ensure pause button is easily tappable and not too close to screen edge
6. Prevent accidental double-tap zoom on iOS Safari
7. Add viewport meta tag in index.html: `<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">`
8. Handle orientation: game works in both portrait and landscape (portrait preferred)

**Acceptance Criteria:**
- [ ] Touch drag moves player smoothly on mobile
- [ ] No browser scroll/zoom during gameplay
- [ ] All buttons are minimum 44x44px
- [ ] UI text is readable on 320px-width screens
- [ ] Game plays correctly in portrait and landscape
- [ ] No iOS Safari double-tap zoom issues
- [ ] Viewport meta tag prevents unwanted scaling
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 17
**Refs:** SPECIFICATION.md §7.3; IMPLEMENTATION.md §6.2

---

## Phase 6: Testing & Release

> Final testing, CI setup, documentation, and production build.
> After this phase: project is ready for deployment.

### Task 19: Unit Tests & Linting

**Write comprehensive unit tests and ensure full lint compliance.**

**Files to create:**
- `tests/managers/ScoreManager.test.ts` -- If not already complete: full coverage
- `tests/managers/DifficultyManager.test.ts` -- If not already complete: full coverage
- `tests/managers/StorageManager.test.ts` -- If not already complete: full coverage
- `tests/config/DifficultyConfig.test.ts` -- Verify preset values match SPEC §3.4.1

**Files to modify:**
- All `src/` files -- Fix any ESLint warnings/errors

**Test cases to cover:**
1. ScoreManager: increment, double points, format with commas, reset
2. DifficultyManager: initial values per preset, speed increase per tick, cap at max speed, spawn rate decrease
3. StorageManager: load defaults, save/load cycle, corrupted data handling, high score update logic
4. DifficultyConfig: preset values match SPECIFICATION.md §3.4.1 table exactly

**Acceptance Criteria:**
- [ ] `npm run test` passes with all tests green
- [ ] Manager test coverage > 80% on business logic
- [ ] `npm run lint` passes with zero warnings
- [ ] `npx tsc --noEmit` passes
- [ ] No `any` types in source code

**Dependencies:** Task 18
**Refs:** IMPLEMENTATION.md §9

---

### Task 20: Documentation & Production Build

**Write project documentation and verify production build.**

**Files to create/modify:**
- `README.md` -- Complete documentation: description, features, setup, build, deploy, tech stack, license
- `.github/workflows/ci.yml` -- GitHub Actions CI pipeline (lint, typecheck, test, build)

**Implementation:**
1. README.md sections:
   - Project description and screenshot/gif placeholder
   - Features list
   - Tech stack table
   - Prerequisites (Node.js 20+)
   - Installation: `npm install`
   - Development: `npm run dev`
   - Build: `npm run build`
   - Deployment instructions (copy dist/ to web server)
   - Controls (keyboard, mouse, touch)
   - License (MIT)
2. CI pipeline:
   - Trigger: push to main, pull requests
   - Steps: checkout, setup Node 20, npm ci, lint, typecheck, test, build
   - Cache node_modules
3. Verify production build:
   - `npm run build` succeeds
   - `npm run preview` serves build output correctly
   - Total dist/ size < 5MB
   - Game plays correctly from built assets

**Acceptance Criteria:**
- [ ] README.md is complete with all sections
- [ ] `npm run build` produces dist/ directory
- [ ] `npm run preview` serves playable game
- [ ] dist/ total size < 5MB
- [ ] CI workflow file is valid YAML
- [ ] All CI steps would pass (lint, typecheck, test, build)
- [ ] No emojis in README (per project rules)
- [ ] `npx tsc --noEmit` passes

**Dependencies:** Task 19
**Refs:** IMPLEMENTATION.md §11, §12; SPECIFICATION.md §9, §10.2

---

## Milestones

| Milestone          | After Task | What's Achieved                               | Demo-able?              |
|--------------------|-----------|-----------------------------------------------|-------------------------|
| Foundation         | Task 3    | Project builds, Phaser canvas renders          | Empty canvas + loading  |
| Building Blocks    | Task 10   | All entities and managers implemented          | Unit test results       |
| Playable Game      | Task 14   | Full game loop with placeholder graphics       | Play the game           |
| Visual Complete    | Task 16   | Pixel art + audio integrated                  | Beautiful game          |
| Polished           | Task 18   | Effects, animations, mobile-ready             | Polished game           |
| Release            | Task 20   | Tests, docs, CI, production build             | Ship it                 |

---

## Dependency Graph

```
[T1] --> [T2] --> [T3]
              |
              +--> [T4] --> [T9]
              |         |
              |         +--> [T10]
              |
              +--> [T5] ----+
              |              |
              +--> [T6] ----+--> [T12] --> [T13]
              |              |         |
              +--> [T7] ----+         +--> [T14] --> [T15] --> [T17] --> [T18] --> [T19] --> [T20]
              |              |                   |
              +--> [T8] ----+                   +--> [T16] ---^
              |
              +--> [T11] (parallel with T5-T10, needs T3+T4+T10)
```

**Parallelizable groups:**
- Tasks 4, 5, 6, 7, 8 can run in parallel after Task 2
- Tasks 15 and 16 can run in parallel after Task 14
