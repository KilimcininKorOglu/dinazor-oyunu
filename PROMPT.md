# DinazorKac -- Claude Code Implementation Prompt

## Project Overview

DinazorKac, tarayici tabanli bir 2D arcade oyunudur. Oyuncu, ekranin alt kisminda bulunan bir dinozoru klavye, mouse veya dokunmatik kontroller ile saga-sola hareket ettirerek yukaridan surekli dusen meteorlardan kacmaya calisir. Chrome'un cevrimdisi dinozor oyunundan esinlenmistir.

Oyun Phaser 4.0 ile gelistirilir. Renkli pixel art gorunumune, tam ses destegine, uc zorluk seviyesine, power-up sistemine, yerel skor tablosuna ve tam mobil uyumluluga sahiptir. TypeScript ile yazilir, Vite ile build edilir. Acik kaynak, MIT lisanslidir.

## Tech Stack

| Layer      | Technology         | Version |
|------------|-------------------|---------|
| Language   | TypeScript        | 5.7+    |
| Runtime    | Browser (ES2020+) | --      |
| Framework  | Phaser            | 4.0.0   |
| Build      | Vite              | 8.x     |
| Testing    | Vitest            | 3.x     |
| Linting    | ESLint            | 9.x     |
| Formatter  | Prettier          | 3.x     |

## Project Structure

> **Working directory:** The project folder already exists and is your CWD. Do NOT
> create a wrapper folder. Do NOT `cd` into a subfolder. All paths run relative to
> CWD. The planning documents (SPECIFICATION.md, IMPLEMENTATION.md, TASKS.md,
> BRANDING.md, PROMPT.md) live at the CWD root -- leave them in place.

```
.
├── public/
│   └── favicon.ico
├── src/
│   ├── main.ts
│   ├── config/
│   │   ├── GameConfig.ts
│   │   ├── DifficultyConfig.ts
│   │   └── AssetKeys.ts
│   ├── scenes/
│   │   ├── BootScene.ts
│   │   ├── MenuScene.ts
│   │   ├── GameScene.ts
│   │   └── GameOverScene.ts
│   ├── entities/
│   │   ├── Player.ts
│   │   ├── Meteor.ts
│   │   └── PowerUp.ts
│   ├── managers/
│   │   ├── ScoreManager.ts
│   │   ├── AudioManager.ts
│   │   ├── InputManager.ts
│   │   ├── DifficultyManager.ts
│   │   ├── PowerUpManager.ts
│   │   ├── MeteorSpawner.ts
│   │   └── StorageManager.ts
│   ├── ui/
│   │   ├── HUD.ts
│   │   ├── PauseOverlay.ts
│   │   ├── Button.ts
│   │   └── DifficultySelector.ts
│   ├── events/
│   │   └── GameEvents.ts
│   └── types/
│       └── index.ts
├── assets/
│   ├── sprites/
│   │   ├── dino-sheet.png
│   │   ├── meteors.png
│   │   ├── powerups.png
│   │   ├── background.png
│   │   └── ui-atlas.png
│   ├── audio/
│   │   ├── music/
│   │   │   ├── menu-theme.mp3
│   │   │   └── game-theme.mp3
│   │   └── sfx/
│   │       ├── hit.mp3
│   │       ├── powerup-collect.mp3
│   │       ├── shield-break.mp3
│   │       ├── gameover.mp3
│   │       ├── new-record.mp3
│   │       └── button-click.mp3
│   └── fonts/
│       └── pixel-font.png
├── tests/
│   ├── managers/
│   │   ├── ScoreManager.test.ts
│   │   ├── DifficultyManager.test.ts
│   │   └── StorageManager.test.ts
│   └── config/
│       └── DifficultyConfig.test.ts
├── index.html
├── vite.config.ts
├── tsconfig.json
├── package.json
├── .eslintrc.json
├── .prettierrc
├── .gitignore
├── LICENSE
└── README.md
```

## Dependencies

```bash
npm init -y
npm install phaser@4
npm install -D typescript vite vitest eslint prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

## Configuration Files

### tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "declaration": false,
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules", "dist"]
}
```

### vite.config.ts
```typescript
import { defineConfig } from 'vite';
import path from 'path';

export default defineConfig({
  base: './',
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    target: 'es2020',
    minify: 'esbuild',
    sourcemap: false,
  },
  server: {
    port: 3000,
    open: true,
  },
});
```

### .eslintrc.json
```json
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "parserOptions": {
    "ecmaVersion": 2020,
    "sourceType": "module"
  },
  "rules": {
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/explicit-function-return-type": "off",
    "@typescript-eslint/no-non-null-assertion": "warn",
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  },
  "ignorePatterns": ["dist/", "node_modules/"]
}
```

### .prettierrc
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "bracketSpacing": true
}
```

### .gitignore
```
node_modules/
dist/
coverage/
.env
.env.local
*.log
.DS_Store
.vite/
```

### index.html
```html
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta name="description" content="DinazorKac - Meteorlardan kac, hayatta kal!" />
  <title>DinazorKac</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background-color: #1a1a2e;
      overflow: hidden;
      touch-action: none;
      -webkit-tap-highlight-color: transparent;
    }
    #game-container {
      width: 100vw;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
    }
  </style>
</head>
<body>
  <div id="game-container"></div>
  <script type="module" src="/src/main.ts"></script>
</body>
</html>
```

### package.json scripts
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "preview": "vite preview",
    "typecheck": "tsc --noEmit",
    "lint": "eslint src/ --ext .ts",
    "lint:fix": "eslint src/ --ext .ts --fix",
    "format": "prettier --write src/",
    "test": "vitest run",
    "test:watch": "vitest"
  }
}
```

## Game Constants

```typescript
// All game parameters -- use these values throughout:
const GAME_WIDTH = 480;
const GAME_HEIGHT = 720;
const PLAYER_Y_OFFSET = 50;          // from bottom edge
const METEOR_MAX_COUNT = 30;
const SCORE_PER_TICK = 1;            // per 100ms
const POWER_UP_FALL_SPEED = 100;     // px/s
const SHIELD_DURATION = -1;          // infinite (consumed on hit)
const SLOW_MOTION_DURATION = 5000;   // ms
const SLOW_MOTION_FACTOR = 0.5;
const DOUBLE_POINTS_DURATION = 8000; // ms
const MAX_METEOR_SPEED = 600;        // px/s
const DIFFICULTY_TICK_INTERVAL = 10000; // ms
```

## Difficulty Presets

```typescript
// Three presets -- exact values:
const difficultyPresets = {
  easy:   { initialMeteorSpeed: 150, initialSpawnRate: 800, speedIncreaseRate: 0.03, powerUpInterval: 15000, playerSpeed: 400 },
  medium: { initialMeteorSpeed: 250, initialSpawnRate: 600, speedIncreaseRate: 0.05, powerUpInterval: 20000, playerSpeed: 350 },
  hard:   { initialMeteorSpeed: 350, initialSpawnRate: 400, speedIncreaseRate: 0.07, powerUpInterval: 25000, playerSpeed: 300 },
};
```

## Color Palette

```typescript
const Colors = {
  dinoGreen:    0x4ade80,  // Player dinosaur
  meteorFire:   0xf97316,  // Meteors
  powerGold:    0xfbbf24,  // Score, double points
  deepNavy:     0x1a1a2e,  // Background
  darkIndigo:   0x16213e,  // UI panels
  white:        0xffffff,  // Text
  softGray:     0x94a3b8,  // Secondary text
  slate:        0x334155,  // Borders
  shieldCyan:   0x22d3ee,  // Shield power-up
  slowEmerald:  0x10b981,  // Slow motion power-up
  error:        0xef4444,  // Game over, damage
};
```

## localStorage Schema

```typescript
// Key: "dinazorKac"
interface StorageData {
  highScores: { easy: number; medium: number; hard: number };
  settings: {
    difficulty: 'easy' | 'medium' | 'hard';
    musicEnabled: boolean;
    sfxEnabled: boolean;
  };
}
// Default: highScores all 0, difficulty "medium", music+sfx true
```

## Implementation Order

Execute these steps sequentially. Each builds on the previous.

---

### Step 1: Project Scaffolding

**Files:** All config files listed above, all directories with `.gitkeep`

1. Create `package.json` with scripts above
2. Install all dependencies (exact command in Dependencies section)
3. Create `tsconfig.json`, `vite.config.ts`, `.eslintrc.json`, `.prettierrc`, `.gitignore`
4. Create `index.html` with the exact content above
5. Create all directories from the project structure (empty with `.gitkeep`)
6. Create `LICENSE` with MIT license text
7. Create minimal `README.md` (project name + one-line description)

**Checkpoint:** `npx tsc --noEmit` passes. `npm run dev` starts Vite on port 3000.

---

### Step 2: Core Types & Events

**Files:** `src/types/index.ts`, `src/events/GameEvents.ts`, `src/config/AssetKeys.ts`, `src/config/GameConfig.ts`, `src/config/DifficultyConfig.ts`

**Types to define in `src/types/index.ts`:**
```typescript
export type DifficultyLevel = 'easy' | 'medium' | 'hard';
export type MeteorSize = 'small' | 'medium' | 'large';
export type PowerUpType = 'shield' | 'slowMotion' | 'doublePoints';

export interface DifficultyParams {
  initialMeteorSpeed: number;
  initialSpawnRate: number;
  speedIncreaseRate: number;
  powerUpInterval: number;
  playerSpeed: number;
}

export interface StorageData {
  highScores: Record<DifficultyLevel, number>;
  settings: GameSettings;
}

export interface GameSettings {
  difficulty: DifficultyLevel;
  musicEnabled: boolean;
  sfxEnabled: boolean;
}
```

**Events in `src/events/GameEvents.ts`:**
```typescript
export const GameEvents = {
  SCORE_CHANGED:       'score:changed',
  POWER_UP_COLLECTED:  'powerup:collected',
  POWER_UP_EXPIRED:    'powerup:expired',
  GAME_OVER:           'game:over',
  SHIELD_BROKEN:       'shield:broken',
  NEW_HIGH_SCORE:      'score:newHigh',
} as const;
```

**GameConfig in `src/config/GameConfig.ts`:**
```typescript
import Phaser from 'phaser';

export const GAME_WIDTH = 480;
export const GAME_HEIGHT = 720;

export const gameConfig: Phaser.Types.Core.GameConfig = {
  type: Phaser.AUTO,
  parent: 'game-container',
  width: GAME_WIDTH,
  height: GAME_HEIGHT,
  scale: {
    mode: Phaser.Scale.FIT,
    autoCenter: Phaser.Scale.CENTER_BOTH,
  },
  physics: {
    default: 'arcade',
    arcade: { gravity: { x: 0, y: 0 }, debug: false },
  },
  scene: [], // Scenes added in main.ts
  backgroundColor: '#1a1a2e',
};
```

**DifficultyConfig:** Define `difficultyPresets` with exact values from the Difficulty Presets section above. Export game constants (METEOR_MAX_COUNT, SCORE_PER_TICK, etc.).

**AssetKeys:** Define string constants for all sprite and audio asset keys. Organized by category.

**Checkpoint:** `npx tsc --noEmit` passes.

---

### Step 3: Entry Point & Boot Scene

**Files:** `src/main.ts`, `src/scenes/BootScene.ts`

**main.ts:**
```typescript
import Phaser from 'phaser';
import { gameConfig } from './config/GameConfig';
import { BootScene } from './scenes/BootScene';
// Import other scenes as they're created

const config = { ...gameConfig, scene: [BootScene] };
new Phaser.Game(config);
```

**BootScene:**
- `preload()`: Load all assets (use placeholder colors for sprites initially -- create colored rectangles using Phaser.GameObjects.Graphics and generateTexture)
- Display a loading progress bar using Graphics (dark background rect + green fill rect that expands)
- `create()`: After loading, transition to MenuScene
- For initial development, generate colored rectangle textures:
  - Player: green rectangle (32x48)
  - Meteor small: orange circle (24px)
  - Meteor medium: orange circle (40px)
  - Meteor large: orange circle (56px)
  - PowerUp shield: cyan rectangle (20x20)
  - PowerUp slowMotion: green rectangle (20x20)
  - PowerUp doublePoints: gold rectangle (20x20)

**Checkpoint:** `npm run dev` shows loading bar, then blank MenuScene canvas.

---

### Step 4: StorageManager

**Files:** `src/managers/StorageManager.ts`, `tests/managers/StorageManager.test.ts`

```typescript
export class StorageManager {
  private static readonly STORAGE_KEY = 'dinazorKac';

  static load(): StorageData {
    try {
      const raw = localStorage.getItem(this.STORAGE_KEY);
      return raw ? JSON.parse(raw) : this.getDefaults();
    } catch {
      return this.getDefaults();
    }
  }

  static save(data: StorageData): void {
    try { localStorage.setItem(this.STORAGE_KEY, JSON.stringify(data)); }
    catch { /* quota or access error -- silently continue */ }
  }

  static updateHighScore(difficulty: DifficultyLevel, score: number): boolean {
    const data = this.load();
    if (score > data.highScores[difficulty]) {
      data.highScores[difficulty] = score;
      this.save(data);
      return true;
    }
    return false;
  }

  static updateSettings(settings: Partial<GameSettings>): void {
    const data = this.load();
    data.settings = { ...data.settings, ...settings };
    this.save(data);
  }

  static getDefaults(): StorageData {
    return {
      highScores: { easy: 0, medium: 0, hard: 0 },
      settings: { difficulty: 'medium', musicEnabled: true, sfxEnabled: true },
    };
  }
}
```

**Tests:** Load defaults when empty, save/load cycle, corrupted JSON fallback, high score update returns true/false correctly, settings merge works.

---

### Step 5: InputManager

**Files:** `src/managers/InputManager.ts`

Unified input for keyboard (arrows + A/D), mouse, and touch:

```typescript
export class InputManager {
  private cursors!: Phaser.Types.Input.Keyboard.CursorKeys;
  private keyA!: Phaser.Input.Keyboard.Key;
  private keyD!: Phaser.Input.Keyboard.Key;

  setup(scene: Phaser.Scene): void {
    this.cursors = scene.input.keyboard!.createCursorKeys();
    this.keyA = scene.input.keyboard!.addKey('A');
    this.keyD = scene.input.keyboard!.addKey('D');
  }

  getMovementX(scene: Phaser.Scene, playerX: number): number {
    if (this.cursors.left.isDown || this.keyA.isDown) return -1;
    if (this.cursors.right.isDown || this.keyD.isDown) return 1;

    const pointer = scene.input.activePointer;
    if (pointer.isDown) {
      const diff = pointer.x - playerX;
      if (Math.abs(diff) > 10) return Math.sign(diff);
    }
    return 0;
  }
}
```

---

### Step 6: Player Entity

**Files:** `src/entities/Player.ts`

Player extends `Phaser.Physics.Arcade.Sprite`:
- Position: bottom of screen (GAME_HEIGHT - PLAYER_Y_OFFSET)
- Horizontal-only movement, clamped to screen bounds (0 + halfWidth to GAME_WIDTH - halfWidth)
- `update(delta, movementX)`: apply velocity based on speed * movementX
- `hasShield` boolean flag
- `activateShield()`: set flag, add cyan tint overlay
- `deactivateShield()`: clear flag, remove tint, play break effect
- Hitbox: set body size to 80% of sprite dimensions (forgiveness margin)
- Use placeholder green rectangle texture initially

---

### Step 7: Meteor Entity & Spawner

**Files:** `src/entities/Meteor.ts`, `src/managers/MeteorSpawner.ts`

**Meteor** extends `Phaser.Physics.Arcade.Sprite`:
- Three sizes: small (24px), medium (40px), large (56px)
- `activate(speed, size)`: set position to random X at Y=-50, set velocity.y = speed, setVisible(true), setActive(true)
- `preUpdate()`: if y > GAME_HEIGHT + 50, call `deactivate()` (setVisible false, setActive false, setVelocity 0)
- Add rotation tween during fall (continuous spinning)

**MeteorSpawner:**
- Uses `Phaser.Physics.Arcade.Group` with `maxSize: 30` and `classType: Meteor`
- `spawn(x, speed, size)`: get from pool, activate
- Ensure random X has minimum spread: divide screen into 3 zones, alternate spawning zones to prevent clustering
- Track spawn timer, compare against current spawn rate

---

### Step 8: PowerUp Entity & Manager

**Files:** `src/entities/PowerUp.ts`, `src/managers/PowerUpManager.ts`

**PowerUp** extends `Phaser.Physics.Arcade.Sprite`:
- Three types: shield (cyan), slowMotion (green), doublePoints (gold)
- Falls at POWER_UP_FALL_SPEED (100px/s) -- slower than meteors
- Distinct colored rectangles for each type (until pixel art is added)

**PowerUpManager:**
- Spawn timer against difficulty's powerUpInterval
- `trySpawn()`: if timer elapsed, spawn random type at random X
- Track active effects with remaining duration
- `activateEffect(type)`: emit POWER_UP_COLLECTED event
  - shield: call player.activateShield()
  - slowMotion: set slowMotionActive flag, timer = 5000ms
  - doublePoints: set doublePointsActive flag, timer = 8000ms
- `update(delta)`: decrement active timers, expire when done
- Stacking: reset timer, don't multiply (e.g., collecting slowMotion while active resets to 5000ms)
- slowMotion expiry: gradual restore over 1 second (not instant snap)

---

### Step 9: ScoreManager & DifficultyManager

**Files:** `src/managers/ScoreManager.ts`, `src/managers/DifficultyManager.ts`, `tests/managers/ScoreManager.test.ts`, `tests/managers/DifficultyManager.test.ts`

**ScoreManager:**
- Accumulator pattern: track elapsed ms, add SCORE_PER_TICK every 100ms
- `doublePointsActive` flag doubles the increment
- `getScore()`: return integer score
- `getFormattedScore()`: return with thousands separator ("1,234")
- `reset()`: zero score and deactivate multipliers

**DifficultyManager:**
- Constructor receives DifficultyParams preset
- Track elapsed time, every DIFFICULTY_TICK_INTERVAL:
  - `currentSpeed *= (1 + speedIncreaseRate)` capped at MAX_METEOR_SPEED
  - `currentSpawnRate *= (1 - speedIncreaseRate * 0.5)` floored at 200ms
- `getCurrentSpeed()`, `getCurrentSpawnRate()` getters
- `applySlowMotion(factor)` / `removeSlowMotion()` for temporary speed reduction

**Tests:**
- ScoreManager: increment rate, double points, format, reset
- DifficultyManager: initial values match preset, speed increases per tick, speed caps at max, spawn rate decreases

**Checkpoint:** `npm run test` passes all tests.

---

### Step 10: AudioManager

**Files:** `src/managers/AudioManager.ts`

- Hold reference to Phaser.Scene for sound playback
- `playMusic(key)`: play in loop; handle browser autoplay by deferring until first interaction
- `stopMusic()`: stop current music
- `playSfx(key)`: play one-shot sound
- `setMusicEnabled(bool)` / `setSfxEnabled(bool)`: toggle, persist via StorageManager
- Load initial settings from StorageManager
- Guard all play calls: check enabled flag before playing
- Handle AudioContext resume on first user gesture

---

### Step 11: MenuScene

**Files:** `src/scenes/MenuScene.ts`, `src/ui/Button.ts`, `src/ui/DifficultySelector.ts`

**Button class:**
- Phaser.GameObjects.Container with background rect + text
- Interactive: pointerover (lighten), pointerout (reset), pointerdown (darken), pointerup (callback)
- Minimum size: 44x44px (mobile touch target)
- Constructor: (scene, x, y, text, callback, options?)

**DifficultySelector:**
- Three Button-like toggles: "Kolay", "Orta", "Zor"
- Highlight selected (different background color)
- On select: update StorageManager settings, emit change event

**MenuScene layout (480x720 canvas):**
- Y=120: "DinazorKac" title (large white text, pixel font style)
- Y=280: "Oyna" button (green, wide)
- Y=380: DifficultySelector (3 inline toggles)
- Y=460: "En Yuksek: {score}" text (updates when difficulty changes)
- Y=580: Music toggle button (left), SFX toggle button (right)
- Start menu music on scene create
- "Oyna" click: stop music, start GameScene with { difficulty } data

---

### Step 12: GameScene -- Core Loop

**Files:** `src/scenes/GameScene.ts`, `src/ui/HUD.ts`

This is the main gameplay scene. Initialize ALL systems in `create()`:

```typescript
create(): void {
  const difficulty = this.scene.settings.data?.difficulty || 'medium';

  // Initialize managers
  this.inputManager = new InputManager();
  this.inputManager.setup(this);
  this.scoreManager = new ScoreManager();
  this.difficultyManager = new DifficultyManager(difficultyPresets[difficulty]);
  this.audioManager = new AudioManager(this);
  this.meteorSpawner = new MeteorSpawner(this);
  this.powerUpManager = new PowerUpManager(this, difficultyPresets[difficulty]);

  // Create player
  this.player = new Player(this, GAME_WIDTH / 2, GAME_HEIGHT - PLAYER_Y_OFFSET);

  // Create HUD
  this.hud = new HUD(this);

  // Collisions
  this.physics.add.overlap(this.player, this.meteorSpawner.getGroup(), this.onMeteorHit, undefined, this);
  this.physics.add.overlap(this.player, this.powerUpManager.getGroup(), this.onPowerUpCollect, undefined, this);

  // Start music
  this.audioManager.playMusic(AssetKeys.MUSIC_GAME);
}

update(time: number, delta: number): void {
  if (this.isGameOver || this.isPaused) return;

  const moveX = this.inputManager.getMovementX(this, this.player.x);
  this.player.update(delta, moveX);
  this.meteorSpawner.update(delta, this.difficultyManager);
  this.powerUpManager.update(delta, this.player, this.scoreManager);
  this.scoreManager.update(delta);
  this.difficultyManager.update(delta);
  this.hud.update(this.scoreManager, this.powerUpManager);
}
```

**Collision: meteor-player (onMeteorHit):**
1. If player has shield: deactivate shield, destroy meteor, play shield-break SFX, flash player
2. If no shield: set isGameOver=true, play hit SFX, screen flash (white overlay 100ms), camera shake, check high score, transition to GameOverScene with { score, difficulty, isNewRecord }

**Collision: powerup-player (onPowerUpCollect):**
1. Determine type, activate via PowerUpManager
2. Play collect SFX
3. Destroy/deactivate power-up sprite

**HUD (src/ui/HUD.ts):**
- Top-right: "Skor: {formatted}" text
- Top-left: Active power-up icons with timer bars (shrinking rectangles)
- Top-right below score: Pause button (small, mobile-friendly)

**Checkpoint:** Game is fully playable with placeholder graphics. Meteors fall, player moves, collisions work, score increments, power-ups function, game over triggers.

---

### Step 13: Pause System

**Files:** `src/ui/PauseOverlay.ts` (new), modify `src/scenes/GameScene.ts`, `src/managers/InputManager.ts`

- Add Escape key listener in GameScene (and dedicated pause button in HUD)
- On pause: set isPaused=true, stop physics with `this.physics.pause()`, stop all timers
- Show PauseOverlay: dark semi-transparent rectangle over game, 3 buttons:
  - "Devam Et": resume physics, hide overlay, set isPaused=false
  - "Yeniden Basla": restart GameScene with same difficulty
  - "Ana Menu": go to MenuScene

---

### Step 14: GameOverScene

**Files:** `src/scenes/GameOverScene.ts`

Receive from GameScene: `{ score: number, difficulty: DifficultyLevel, isNewRecord: boolean }`

Layout:
- Y=180: "Game Over" text, animated entrance (scale from 0 to 1 with bounce ease)
- Y=300: "Skor: {formatted}" large text
- Y=360: "En Yuksek: {highScore}" smaller text
- If isNewRecord: Y=420: "Yeni Rekor!" gold text with pulsing animation + particle burst
- Y=500: "Tekrar Oyna" button (green)
- Y=570: "Ana Menu" button (gray)
- Play gameover SFX or new-record SFX based on isNewRecord
- Quick restart: after 1 second delay, any key/tap restarts (same difficulty)

**Checkpoint:** Complete game loop: Menu -> Game -> Game Over -> Replay/Menu. All placeholder graphics. Fully playable.

---

### Step 15: Pixel Art Sprites

**Files:** All files in `assets/sprites/`, modify BootScene and entities

Create pixel art assets. If unable to draw actual pixel art, create procedural pixel-style graphics using Phaser.GameObjects.Graphics with:

**Dinosaur (dino-sheet):**
- 32x48 pixel size, green (#4ade80) body
- Simple T-Rex shape: big head, small arms, thick tail, two legs
- 4 frames: idle, walk-left, walk-right, walk-left-alt (2-frame walk cycle)
- Shield frame: same dino with cyan (#22d3ee) glow border

**Meteors (meteors.png):**
- 3 sizes on same sheet: 24x24, 40x40, 56x56
- Orange/red (#f97316, #c2410c) rocky spheres with crater details
- 2-3 variants per size for visual variety

**Power-ups (powerups.png):**
- 20x20 each, 3 types on same sheet
- Shield: cyan circle with "S" shape
- SlowMotion: green hourglass shape
- DoublePoints: gold star shape

**Background (background.png):**
- 480x720, dark navy (#1a1a2e) gradient to slightly lighter at bottom
- Scattered small white/gray dots as stars

**UI Atlas:**
- Pause icon (two vertical bars)
- Speaker icon (on/off variants)
- Button 9-patch or simple rect backgrounds

Update BootScene to load actual sprite sheets with frame dimensions.
Update Player, Meteor, PowerUp entities to use sprite animations.

---

### Step 16: Audio Integration

**Files:** All files in `assets/audio/`, modify BootScene, all scenes

Source or generate audio:
- **Music**: Use royalty-free chiptune loops. Menu theme (calm, 8-bit). Game theme (upbeat, 8-bit).
- **SFX**: Use jsfxr or similar to generate retro sound effects:
  - hit.mp3: explosion/crash sound (~0.5s)
  - powerup-collect.mp3: positive pickup chime (~0.3s)
  - shield-break.mp3: glass break / energy dissipation (~0.4s)
  - gameover.mp3: descending failure tone (~1s)
  - new-record.mp3: triumphant fanfare (~1.5s)
  - button-click.mp3: short click/beep (~0.1s)

All MP3 format for broad browser compatibility.
Wire AudioManager calls to all game events in GameScene, MenuScene, GameOverScene.
Handle browser autoplay: call `this.sound.unlock()` or resume AudioContext on first pointer event.

**Checkpoint:** Game looks and sounds complete. Pixel art sprites, animated dinosaur, distinct meteors and power-ups, background, music, and all sound effects.

---

### Step 17: Visual Effects & Animations

**Modify:** GameScene, GameOverScene, Meteor, Player, HUD

1. **Meteor trail particles**: Phaser ParticleEmitter following each meteor -- small orange/red particles fading out
2. **Screen flash on hit**: White rectangle overlay, alpha 0.7 -> 0, duration 100ms
3. **Player damage flash**: Red tint on player sprite for 200ms
4. **Score milestone pop**: Brief scale-up on score text at 100, 500, 1000, 5000 intervals
5. **New record particles**: Gold particle burst on GameOverScene when isNewRecord
6. **Power-up timer bars**: Horizontal bars under HUD power-up icons that shrink over duration
7. **Slow motion visual**: Blue-ish tint on entire camera (slight camera tint)
8. **Scene transitions**: Camera fade-out (200ms) -> fade-in (200ms) between scenes

---

### Step 18: Mobile Optimization

**Modify:** GameConfig, InputManager, HUD, Button, PauseOverlay, MenuScene, GameOverScene

1. Verify `Scale.FIT` + `CENTER_BOTH` in GameConfig
2. Polish touch controls: player follows finger X position smoothly
3. Prevent default touch behaviors via `touch-action: none` in CSS (already in index.html)
4. All buttons minimum 44x44px
5. Scale text relative to canvas size (use proportional font sizes)
6. Add viewport meta tag `maximum-scale=1.0, user-scalable=no` (already in index.html)
7. Test at 320px, 375px, 414px widths
8. Prevent iOS double-tap zoom with CSS `-webkit-tap-highlight-color: transparent`
9. Game works in both portrait (preferred) and landscape

**Checkpoint:** Game is polished, animated, and plays well on mobile devices.

---

### Step 19: Unit Tests & Lint Compliance

**Files:** All test files in `tests/`, fix any lint issues in `src/`

Write/complete tests for:
- `ScoreManager`: increment, doublePoints, format, reset
- `DifficultyManager`: initial values per preset, progressive increase, max cap
- `StorageManager`: defaults, save/load, corrupted data, high score update
- `DifficultyConfig`: preset values match specification exactly

Run and fix:
- `npm run lint` -- zero warnings
- `npx tsc --noEmit` -- zero errors
- `npm run test` -- all pass

---

### Step 20: Documentation & Production Build

**Files:** `README.md`, `.github/workflows/ci.yml`

**README.md sections (no emojis):**
- Project name and description
- Features list (bullet points)
- Tech stack
- Prerequisites (Node.js 20+)
- Installation and running
- Build and deployment
- Controls (keyboard, mouse, touch)
- License

**CI workflow (.github/workflows/ci.yml):**
```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm run test
      - run: npm run build
```

**Final verification:**
- `npm run build` produces `dist/`
- `npm run preview` serves playable game
- `dist/` total size < 5MB

## Quality Checks

After all steps complete, verify:
- [ ] `npm run lint` passes with 0 warnings
- [ ] `npx tsc --noEmit` passes with 0 errors
- [ ] `npm run test` passes with 0 failures
- [ ] `npm run build` produces dist/ directory
- [ ] `npm run preview` serves a fully playable game
- [ ] Game works on desktop (keyboard + mouse)
- [ ] Game works on mobile (touch controls)
- [ ] Menu -> Game -> Game Over -> Replay flow works
- [ ] All three difficulty levels work correctly
- [ ] All three power-up types function
- [ ] High scores save and persist across sessions
- [ ] Music and SFX play correctly with toggle support
- [ ] Pause and resume work
- [ ] No console errors in browser
- [ ] README.md is complete and professional (no emojis)
