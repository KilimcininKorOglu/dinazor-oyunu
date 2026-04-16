# DinazorKac -- Implementation Plan

> Technical blueprint derived from SPECIFICATION.md.

## 1. Tech Stack

### 1.1 Stack Summary

| Layer      | Technology   | Version | Rationale                                                                                      |
|------------|-------------|---------|------------------------------------------------------------------------------------------------|
| Language   | TypeScript  | 5.7+    | Tip guvenligi, IDE destegi, Phaser ile birinci sinif entegrasyon                                |
| Runtime    | Browser     | ES2020+ | Hedef: modern tarayicilar (Chrome 80+, Firefox 75+, Safari 13+)                                |
| Framework  | Phaser      | 4.0.0   | Olgun 2D oyun framework'u: sprite, fizik, ses, girdi yonetimi dahili. Yeni WebGL renderer.     |
| Build      | Vite        | 8.x     | Hizli dev server, HMR, Rolldown tabanli build, TypeScript destegi dahili                       |
| Testing    | Vitest      | 3.x     | Vite ile native entegrasyon, hizli, TypeScript-first                                           |
| Linting    | ESLint      | 9.x     | Flat config, TypeScript parser, kod kalitesi standartlari                                      |
| Formatter  | Prettier    | 3.x     | Tutarli kod formatlamasi                                                                       |

### 1.2 Key Technical Decisions

#### Decision: Phaser 4.0 vs Vanilla Canvas vs PixiJS

- **Context**: SPECIFICATION.md, 2D arcade oyunu, sprite animasyonlar, carpisma algilama, ses yonetimi, girdi islemcisi (Ref: SPEC 4.1)
- **Options Considered**:
  1. **Vanilla Canvas**: Tam kontrol, sifir bagimlilik / Tum altyapiyi sifirdan yazmak gerekir
  2. **PixiJS**: Yuksek performansli render / Oyun mantigi, fizik, ses ayrica yazilmali
  3. **Phaser 4.0**: Sprite, fizik, ses, girdi, scene yonetimi dahili / Daha buyuk bundle boyutu
- **Choice**: Phaser 4.0
- **Rationale**: Arcade fizigi (carpisma algilama), scene yonetimi (menu/game/gameover), dahili ses sistemi ve input handler tam bu proje icin. v4.0'in yeni WebGL renderer'i performans avantaji saglar.
- **Consequences**: ~300KB minified bundle eklentisi. Kabul edilebilir.

#### Decision: TypeScript vs JavaScript

- **Context**: Kod guvenligi, IDE destegi, buyuyen kod tabani (Ref: SPEC 4.1)
- **Options Considered**:
  1. **JavaScript**: Sifir kurulum, hizli baslangic / Tip guveniligi yok
  2. **TypeScript**: Tip guvenligi, refactor kolayligi / Ek build adimi
- **Choice**: TypeScript
- **Rationale**: Phaser'in TypeScript type definition'lari mukemmel. Config objeleri, entity tipleri ve event handler'lari icin tip guvenligi hata oranini dusurur.
- **Consequences**: `tsconfig.json` yapilandirmasi, Vite TypeScript entegrasyonu (dahili).

#### Decision: Object Pooling vs Dynamic Creation

- **Context**: Meteor ve power-up'lar surekli olusturulup yok ediliyor (Ref: SPEC 10.3)
- **Options Considered**:
  1. **Dynamic Creation**: Her meteor icin `new` + `destroy()` / GC baskisi, frame drop riski
  2. **Object Pooling**: On-olusturulmus nesne havuzu, yeniden kullanim / Biraz daha karmasik kod
- **Choice**: Object Pooling (Phaser'in dahili Group pool ozelligi)
- **Rationale**: SPEC 10.3 acikca object pooling oneriyor. Phaser'in `Group` sinifi `maxSize` ve `createCallback` ile dahili pool yonetimi sunar.
- **Consequences**: Meteor ve PowerUp gruplari `maxSize` ile sinirlanir. Pool bos kaldiginda yeni nesne olusturulmaz.

#### Decision: Asset Formati

- **Context**: Pixel art gorseller, ses dosyalari (Ref: SPEC 3.6, 10.2)
- **Options Considered**:
  1. **Bireysel PNG'ler**: Basit / Cok HTTP istegi
  2. **Sprite Atlas (Texture Packer)**: Tek dosyada tum sprite'lar / Build adimi gerekir
  3. **Sprite Sheet**: Izgarali sprite dizisi / Basit, Phaser ile kolay
- **Choice**: Sprite Sheet + Sprite Atlas (karma)
- **Rationale**: Karakter animasyonlari icin sprite sheet, UI elemanlari icin atlas. Toplam asset boyutunu 5MB altinda tutar (SPEC 10.2).
- **Consequences**: Asset pipeline'inda TexturePacker veya benzeri arac kullanilabilir ama zorunlu degil.

### 1.3 Dependency Inventory

**Bagimlilik Felsefesi**: Curated minimal -- Phaser cogu isi ustlenir, ek bagimlilik minimumda tutulur.

| Package           | Purpose                                  | License | Justification                                         |
|-------------------|------------------------------------------|---------|-------------------------------------------------------|
| phaser            | 2D oyun framework'u                      | MIT     | Temel framework -- sprite, fizik, ses, girdi           |
| typescript        | Tip guvenligi                            | Apache  | Gelistirme zamani tip kontrolu                        |
| vite              | Build tool + dev server                  | MIT     | HMR, TypeScript destegi, hizli build                  |
| vitest            | Test framework                           | MIT     | Vite native entegrasyon                               |
| eslint            | Linting                                  | MIT     | Kod kalite standartlari                               |
| prettier          | Formatter                                | MIT     | Tutarli formatlama                                    |
| @typescript-eslint| TypeScript ESLint parser                 | MIT     | TS dosyalari icin linting                             |

## 2. Design Patterns

### 2.1 Architectural Pattern: Scene-Based Architecture (Phaser Scenes)

**Why:** Oyunun farkli ekranlari (menu, oyun, game over) net sekilde ayrilmali (SPEC 7.2). Phaser'in Scene sistemi tam bu amaca hizmet eder.

**Application:** Her ekran bir Phaser Scene olarak tanimlanir. Scene'ler arasi gecis Phaser'in dahili scene manager'i ile yapilir.

**Code Sketch:**
```typescript
// src/scenes/GameScene.ts
export class GameScene extends Phaser.Scene {
  private player!: Player;
  private meteorGroup!: Phaser.Physics.Arcade.Group;
  private scoreManager!: ScoreManager;

  constructor() {
    super({ key: 'GameScene' });
  }

  create(): void {
    this.player = new Player(this);
    this.meteorGroup = this.physics.add.group({ maxSize: 30 });
    this.scoreManager = new ScoreManager(this);
    this.setupCollisions();
  }

  update(time: number, delta: number): void {
    this.player.update(delta);
    this.scoreManager.update(delta);
  }
}
```

### 2.2 State Machine Pattern (Game States)

**Why:** Oyun durumu (playing, paused, gameOver) kontrol edilmeli; gecersiz gecisler onlenmeli (SPEC 3.2.4).

**Code Sketch:**
```typescript
// src/managers/GameStateManager.ts
type GameState = 'menu' | 'playing' | 'paused' | 'gameOver';

const validTransitions: Record<GameState, GameState[]> = {
  menu:     ['playing'],
  playing:  ['paused', 'gameOver'],
  paused:   ['playing', 'menu'],
  gameOver: ['playing', 'menu'],
};

export class GameStateManager {
  private state: GameState = 'menu';

  transition(to: GameState): boolean {
    if (!validTransitions[this.state].includes(to)) return false;
    this.state = to;
    return true;
  }

  getState(): GameState { return this.state; }
}
```

### 2.3 Observer Pattern (Event Bus)

**Why:** Skor degisikligi, power-up toplama, game over gibi olaylar birden fazla sistemi etkiler (SPEC 4.2). Siki baglanma yerine event-driven iletisim.

**Code Sketch:**
```typescript
// src/events/GameEvents.ts
export const GameEvents = {
  SCORE_CHANGED:    'score:changed',
  POWER_UP_COLLECTED: 'powerup:collected',
  POWER_UP_EXPIRED: 'powerup:expired',
  GAME_OVER:        'game:over',
  SHIELD_BROKEN:    'shield:broken',
  NEW_HIGH_SCORE:   'score:newHigh',
} as const;

// Kullanim -- Phaser'in dahili event emitter'i:
// this.events.emit(GameEvents.SCORE_CHANGED, { score: 1234 });
// this.events.on(GameEvents.SCORE_CHANGED, (data) => this.updateHUD(data));
```

### 2.4 Object Pool Pattern (Meteor & PowerUp Recycling)

**Why:** Surekli olusturulan/yok edilen meteorlar GC baskisi olusturur (SPEC 10.3).

**Code Sketch:**
```typescript
// Phaser'in dahili Group pool ozelligi
export class MeteorSpawner {
  private pool: Phaser.Physics.Arcade.Group;

  constructor(scene: Phaser.Scene) {
    this.pool = scene.physics.add.group({
      classType: Meteor,
      maxSize: 30,
      runChildUpdate: true,
    });
  }

  spawn(x: number, speed: number, size: MeteorSize): void {
    const meteor = this.pool.get(x, -50) as Meteor;
    if (meteor) {
      meteor.activate(speed, size);
    }
  }
}
```

### 2.5 Strategy Pattern (Difficulty Configuration)

**Why:** Uc zorluk seviyesi farkli parametrelere sahip (SPEC 3.4.1). Strateji deseni ile temiz ayrim.

**Code Sketch:**
```typescript
// src/config/DifficultyConfig.ts
export interface DifficultyParams {
  initialMeteorSpeed: number;
  initialSpawnRate: number;
  speedIncreaseRate: number;
  powerUpInterval: number;
  playerSpeed: number;
}

export const difficultyPresets: Record<string, DifficultyParams> = {
  easy:   { initialMeteorSpeed: 150, initialSpawnRate: 800, speedIncreaseRate: 0.03, powerUpInterval: 15000, playerSpeed: 400 },
  medium: { initialMeteorSpeed: 250, initialSpawnRate: 600, speedIncreaseRate: 0.05, powerUpInterval: 20000, playerSpeed: 350 },
  hard:   { initialMeteorSpeed: 350, initialSpawnRate: 400, speedIncreaseRate: 0.07, powerUpInterval: 25000, playerSpeed: 300 },
};
```

### 2.6 Factory Pattern (Entity Creation)

**Why:** Meteor ve power-up olusturma mantigi boyut, tip ve hiz hesaplama icerir. Merkezi factory ile tutarlilik saglanir.

**Code Sketch:**
```typescript
// src/entities/PowerUpFactory.ts
export type PowerUpType = 'shield' | 'slowMotion' | 'doublePoints';

export class PowerUpFactory {
  static create(scene: Phaser.Scene, type: PowerUpType, x: number): PowerUp {
    const config = powerUpConfigs[type];
    const powerUp = new PowerUp(scene, x, -30, config.texture, type);
    powerUp.setDuration(config.duration);
    powerUp.setFallSpeed(config.fallSpeed);
    return powerUp;
  }
}
```

## 3. Project Structure

### 3.1 Directory Layout

```
.                                   # CWD = proje koku
├── public/                         # Statik dosyalar (Vite tarafindan oldugu gibi sunulur)
│   └── favicon.ico                 # Site ikonu
├── src/                            # Kaynak kodu
│   ├── main.ts                     # Uygulama giris noktasi, Phaser oyun yapilandirmasi
│   ├── config/                     # Yapilandirma dosyalari
│   │   ├── GameConfig.ts           # Phaser oyun konfigurasyonu (boyut, fizik, sahne listesi)
│   │   ├── DifficultyConfig.ts     # Zorluk seviyeleri parametreleri
│   │   └── AssetKeys.ts            # Asset anahtar sabitleri (sprite, ses isimleri)
│   ├── scenes/                     # Phaser sahneleri
│   │   ├── BootScene.ts            # Asset yukleme + yuklenme cubugu
│   │   ├── MenuScene.ts            # Ana menu ekrani
│   │   ├── GameScene.ts            # Ana oyun sahnesi
│   │   └── GameOverScene.ts        # Game Over ekrani
│   ├── entities/                   # Oyun varliklari (sprite siniflari)
│   │   ├── Player.ts               # Dinozor oyuncu sinifi
│   │   ├── Meteor.ts               # Meteor sprite sinifi
│   │   └── PowerUp.ts              # Power-up sprite sinifi
│   ├── managers/                   # Oyun yoneticileri
│   │   ├── ScoreManager.ts         # Skor hesaplama ve kayit
│   │   ├── AudioManager.ts         # Ses ve muzik yonetimi
│   │   ├── InputManager.ts         # Klavye/mouse/dokunmatik girdi birlestirici
│   │   ├── DifficultyManager.ts    # Zamanla artan zorluk hesaplayici
│   │   ├── PowerUpManager.ts       # Power-up spawn ve etki yonetimi
│   │   ├── MeteorSpawner.ts        # Meteor olusturma ve pool yonetimi
│   │   └── StorageManager.ts       # localStorage okuma/yazma
│   ├── ui/                         # HUD ve menu bilesenleri
│   │   ├── HUD.ts                  # Oyun ici gosterge paneli
│   │   ├── PauseOverlay.ts         # Duraklatma katmani
│   │   ├── Button.ts               # Yeniden kullanilabilir buton sinifi
│   │   └── DifficultySelector.ts   # Zorluk secici bileseni
│   ├── events/                     # Olay tanimlari
│   │   └── GameEvents.ts           # Olay sabit tanimlari
│   └── types/                      # TypeScript tip tanimlari
│       └── index.ts                # Tum paylasilan tipler
├── assets/                         # Oyun asset'leri (Vite import ile dahil edilir)
│   ├── sprites/                    # Sprite sheet'ler ve tekil gorseller
│   │   ├── dino-sheet.png          # Dinozor animasyon sprite sheet
│   │   ├── meteors.png             # Meteor cesitleri sprite sheet
│   │   ├── powerups.png            # Power-up ikonlari
│   │   ├── background.png          # Arka plan gorseli
│   │   └── ui-atlas.png            # UI elemanlar atlas
│   ├── audio/                      # Ses dosyalari
│   │   ├── music/                  # Arka plan muzikleri
│   │   │   ├── menu-theme.mp3      # Menu muzigi
│   │   │   └── game-theme.mp3      # Oyun muzigi
│   │   └── sfx/                    # Ses efektleri
│   │       ├── hit.mp3             # Carpisma sesi
│   │       ├── powerup-collect.mp3 # Power-up toplama
│   │       ├── shield-break.mp3    # Kalkan kirilmasi
│   │       ├── gameover.mp3        # Game over sesi
│   │       ├── new-record.mp3      # Yeni rekor sesi
│   │       └── button-click.mp3    # Buton tiklama
│   └── fonts/                      # Pixel art fontlar
│       └── pixel-font.png          # Bitmap font sprite sheet
├── tests/                          # Test dosyalari
│   ├── managers/                   # Manager unit testleri
│   │   ├── ScoreManager.test.ts    # Skor hesaplama testleri
│   │   ├── DifficultyManager.test.ts
│   │   └── StorageManager.test.ts
│   └── config/                     # Config testleri
│       └── DifficultyConfig.test.ts
├── index.html                      # HTML giris dosyasi
├── vite.config.ts                  # Vite yapilandirmasi
├── tsconfig.json                   # TypeScript yapilandirmasi
├── package.json                    # Proje bagimliliklari ve script'ler
├── .eslintrc.json                  # ESLint yapilandirmasi
├── .prettierrc                     # Prettier yapilandirmasi
├── .gitignore                      # Git ignore kurallari
├── LICENSE                         # MIT lisansi
└── README.md                       # Proje dokumantasyonu
```

**Structural Philosophy:**
- **Feature-based gruplama**: `entities/`, `managers/`, `scenes/` gibi sorumluluk tabanli dizinler
- **Tek sorumluluk**: Her dosya tek bir sinif veya modul icerir
- **Asset ayirimi**: Gorseller, sesler ve fontlar `assets/` altinda kategorize edilir
- **Test eslesmesi**: Test dosyalari `tests/` altinda kaynak dosya yapisini yansitir

### 3.2 Module Breakdown

#### Module: Scenes

- **Path**: `src/scenes/`
- **Responsibility**: Oyun akisinin her asamasini yoneten Phaser Scene'leri
- **Exports**: `BootScene`, `MenuScene`, `GameScene`, `GameOverScene`
- **Imports**: entities, managers, ui, config, events
- **Key Files**:
  - `BootScene.ts` -- Asset yukleme ve ilerleme cubugu
  - `MenuScene.ts` -- Ana menu, zorluk secimi, ayarlar
  - `GameScene.ts` -- Oyun dongusu, carpisma, spawn yonetimi
  - `GameOverScene.ts` -- Sonuc ekrani, yeni rekor kontrolu

#### Module: Entities

- **Path**: `src/entities/`
- **Responsibility**: Oyun ici sprite nesnelerinin davranislarini tanimlar
- **Exports**: `Player`, `Meteor`, `PowerUp`
- **Imports**: config, types
- **Key Files**:
  - `Player.ts` -- Dinozor hareketi, kalkan durumu, hitbox
  - `Meteor.ts` -- Dusme hareketi, boyut cesitleri, pool yapilandirmasi
  - `PowerUp.ts` -- Dusme hareketi, tip bazli gorsel, toplama davranisi

#### Module: Managers

- **Path**: `src/managers/`
- **Responsibility**: Oyun sistemi yoneticileri (skor, ses, girdi, zorluk, depolama)
- **Exports**: `ScoreManager`, `AudioManager`, `InputManager`, `DifficultyManager`, `PowerUpManager`, `MeteorSpawner`, `StorageManager`
- **Imports**: config, events, types
- **Key Files**:
  - `ScoreManager.ts` -- Puan hesabi, high score karsilastirmasi
  - `AudioManager.ts` -- Muzik/SFX oynatma, ses seviyesi kontrolu
  - `InputManager.ts` -- Klavye, mouse, dokunmatik girdi birlestirme
  - `DifficultyManager.ts` -- Zamanla artan zorluk hesaplayici
  - `PowerUpManager.ts` -- Power-up zamanlama, etki yonetimi
  - `MeteorSpawner.ts` -- Meteor olusturma, pool yonetimi
  - `StorageManager.ts` -- localStorage CRUD islemleri

#### Module: UI

- **Path**: `src/ui/`
- **Responsibility**: Oyun ici HUD ve menu bileseleri
- **Exports**: `HUD`, `PauseOverlay`, `Button`, `DifficultySelector`
- **Imports**: managers, events, config

#### Module: Config

- **Path**: `src/config/`
- **Responsibility**: Tum oyun sabitlerini ve yapilandirmalarini merkezi olarak tanimlar
- **Exports**: `GameConfig`, `DifficultyConfig`, `difficultyPresets`, `AssetKeys`
- **Imports**: Yok (leaf module)

#### Module: Events

- **Path**: `src/events/`
- **Responsibility**: Olay adlarini sabit olarak tanimlar
- **Exports**: `GameEvents`
- **Imports**: Yok (leaf module)

#### Module: Types

- **Path**: `src/types/`
- **Responsibility**: Paylasilan TypeScript tip tanimlari
- **Exports**: `GameState`, `DifficultyLevel`, `MeteorSize`, `PowerUpType`, `HighScoreData`, `GameSettings`
- **Imports**: Yok (leaf module)

### 3.3 Module Dependency Graph

```
[Scenes] --> [Entities] --> [Types]
   |              |
   +-------> [Managers] --> [Config]
   |              |              |
   +-------> [UI] ---------> [Events]
                  |
                  +---------> [Types]
```

- `Config`, `Events`, `Types` leaf module'lerdir -- hicbir sey import etmezler
- `Managers` yalnizca `Config`, `Events`, `Types`'a bagimlidir
- `Scenes` her seyi bir araya getirir -- en ust katman

## 4. Data Layer

### 4.1 Runtime State (In-Memory)

Veritabani yoktur. Tum oyun durumu Phaser Scene'lerinin bellekinde tutulur. Yalnizca kalici veriler icin localStorage kullanilir.

### 4.2 localStorage Schema

```typescript
// src/types/index.ts
export interface StorageData {
  highScores: {
    easy: number;
    medium: number;
    hard: number;
  };
  settings: {
    difficulty: DifficultyLevel;
    musicEnabled: boolean;
    sfxEnabled: boolean;
  };
}

// localStorage key: "dinazorKac"
// Deger: JSON.stringify(StorageData)
```

### 4.3 Data Access Pattern

```typescript
// src/managers/StorageManager.ts
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
    try {
      localStorage.setItem(this.STORAGE_KEY, JSON.stringify(data));
    } catch {
      // localStorage dolu veya erisim engelli -- sessizce devam et
    }
  }

  static updateHighScore(difficulty: DifficultyLevel, score: number): boolean {
    const data = this.load();
    if (score > data.highScores[difficulty]) {
      data.highScores[difficulty] = score;
      this.save(data);
      return true; // Yeni rekor
    }
    return false;
  }

  private static getDefaults(): StorageData {
    return {
      highScores: { easy: 0, medium: 0, hard: 0 },
      settings: { difficulty: 'medium', musicEnabled: true, sfxEnabled: true },
    };
  }
}
```

## 6. Frontend Implementation

### 6.1 Scene Architecture

```
BootScene (asset yukleme)
    |
    v
MenuScene (ana menu)
    |
    v
GameScene (oyun dongusu)
    |          |
    v          v
PauseOverlay  GameOverScene
                   |
                   v
              GameScene (tekrar) veya MenuScene
```

### 6.2 Responsive Canvas Strategy

```typescript
// src/config/GameConfig.ts
export const gameConfig: Phaser.Types.Core.GameConfig = {
  type: Phaser.AUTO,           // WebGL varsa WebGL, yoksa Canvas
  parent: 'game-container',
  width: 480,
  height: 720,
  scale: {
    mode: Phaser.Scale.FIT,    // Pencereye sigdir, oran koru
    autoCenter: Phaser.Scale.CENTER_BOTH,
  },
  physics: {
    default: 'arcade',
    arcade: {
      gravity: { x: 0, y: 0 }, // Yercekimi yok -- meteorlar kendi hizlariyla duser
      debug: false,
    },
  },
  scene: [BootScene, MenuScene, GameScene, GameOverScene],
  backgroundColor: '#1a1a2e',
};
```

- **Tasarim boyutu**: 480x720 (9:16 dikey oran -- mobil oncelikli)
- **Olcekleme**: `Phaser.Scale.FIT` ile pencereye sigdirilir
- **Otomatik ortalama**: `CENTER_BOTH` ile her zaman ortada
- **Dokunmatik**: Phaser'in `input.addPointer()` ile coklu dokunma destegi

### 6.3 Input System

```typescript
// src/managers/InputManager.ts
export class InputManager {
  private cursors!: Phaser.Types.Input.Keyboard.CursorKeys;
  private wasd!: { a: Phaser.Input.Keyboard.Key; d: Phaser.Input.Keyboard.Key };
  private pointer!: Phaser.Input.Pointer;

  setup(scene: Phaser.Scene): void {
    // Klavye
    this.cursors = scene.input.keyboard!.createCursorKeys();
    this.wasd = {
      a: scene.input.keyboard!.addKey('A'),
      d: scene.input.keyboard!.addKey('D'),
    };

    // Mouse + Dokunmatik
    this.pointer = scene.input.activePointer;
  }

  getMovementX(playerX: number): number {
    // Klavye onceligi
    if (this.cursors.left.isDown || this.wasd.a.isDown) return -1;
    if (this.cursors.right.isDown || this.wasd.d.isDown) return 1;

    // Dokunmatik/mouse -- pointer aktifse
    if (this.pointer.isDown) {
      const diff = this.pointer.x - playerX;
      if (Math.abs(diff) > 10) return Math.sign(diff);
    }

    return 0;
  }
}
```

### 6.4 Styling

- **Canvas icerigi**: Tum gorsel elemanlar Phaser sprite/text olarak render edilir (DOM CSS yok)
- **HTML shell**: Minimal `index.html` -- sadece `<div id="game-container">` ve arka plan rengi
- **Font**: Bitmap pixel font (Phaser BitmapText) -- web font bagimliligini onler

## 7. Error Handling Strategy

### 7.1 Error Classification

| Category             | Example                       | Handling                            |
|----------------------|-------------------------------|-------------------------------------|
| Asset load failure   | Sprite sheet 404              | Boot ekraninda hata mesaji          |
| localStorage hatasi  | Quota dolu, erisim engelli    | Varsayilan degerler, sessiz fallback |
| Audio playback       | Tarayici otomatik calistirmayI engelledi | Ilk tiklamada baslat   |
| WebGL context loss   | GPU kaynak yetersizligi       | Phaser dahili context restoration   |
| Performans dususu    | FPS < 30                      | Spawn rate gecici azaltma           |

### 7.2 Error Propagation

Oyun istemci taraflidir -- hata propagasyonu basittir:
1. Manager/entity hatasi --> Scene'de yakalanir
2. Kritik hatalar (asset yukleme) --> Kullaniciya gorsel mesaj
3. Kritik olmayan hatalar (ses, depolama) --> Sessiz fallback, konsol uyarisi

## 8. Configuration

### 8.1 Game Constants

| Key                     | Type   | Default | Description                               |
|-------------------------|--------|---------|-------------------------------------------|
| GAME_WIDTH              | number | 480     | Tasarim genisligi (piksel)                |
| GAME_HEIGHT             | number | 720     | Tasarim yuksekligi (piksel)               |
| PLAYER_Y_OFFSET         | number | 50      | Oyuncunun alt kenardan yuksekligi         |
| METEOR_MAX_COUNT        | number | 30      | Ayni anda maksimum meteor                 |
| SCORE_PER_TICK          | number | 1       | Her 100ms'de kazanilan puan               |
| POWER_UP_FALL_SPEED     | number | 100     | Power-up dusme hizi (px/s)               |
| SHIELD_DURATION         | number | -1      | Sinirsiz (carpisma ile biter)             |
| SLOW_MOTION_DURATION    | number | 5000    | Yavaslatma suresi (ms)                    |
| SLOW_MOTION_FACTOR      | number | 0.5     | Yavaslatma carpani                        |
| DOUBLE_POINTS_DURATION  | number | 8000    | Cift puan suresi (ms)                     |
| MAX_METEOR_SPEED        | number | 600     | Maksimum meteor hizi (px/s)              |
| DIFFICULTY_TICK_INTERVAL| number | 10000   | Zorluk artis periyodu (ms)               |

## 9. Testing Strategy

### 9.1 Test Pyramid

| Level       | Tool   | Scope                                      | Target                         |
|-------------|--------|--------------------------------------------|--------------------------------|
| Unit        | Vitest | Manager'lar, config, utility fonksiyonlari | %80+ kapsama (is mantigi)      |
| Integration | Vitest | StorageManager + localStorage mock         | Veri kaliciligi dogrulama       |

Not: Phaser Scene'lerin gorsel/interaktif testleri manual olarak yapilir. E2E test framework'u (Playwright) bu kapsamda yoktur.

### 9.2 Test Patterns

```typescript
// tests/managers/ScoreManager.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { ScoreManager } from '../../src/managers/ScoreManager';

describe('ScoreManager', () => {
  let scoreManager: ScoreManager;

  beforeEach(() => {
    scoreManager = new ScoreManager();
  });

  it('should increase score by 1 per tick', () => {
    scoreManager.update(100); // 100ms delta
    expect(scoreManager.getScore()).toBe(1);
  });

  it('should double score when double points is active', () => {
    scoreManager.activateDoublePoints();
    scoreManager.update(100);
    expect(scoreManager.getScore()).toBe(2);
  });
});
```

### 9.3 CI Pipeline

```
Push/PR --> Lint (ESLint) --> Type Check (tsc --noEmit) --> Unit Tests (Vitest) --> Build (Vite)
```

## 10. Security Implementation

### 10.1 Client-Side Security

Bu tamamen istemci tarafli bir oyundur. Sunucu tarafi guvenlik endisesi yoktur. Ancak:

- **XSS onleme**: Kullanici girdisi alinmaz (oyuncu adi, metin girisi yok)
- **localStorage manipulasyonu**: High score istemci tarafinda saklanir -- anti-cheat bu kapsamda yoktur. Gelecekte sunucu tarafli dogrulama eklenebilir.
- **CSP Headers**: Deploy edildigi sunucuda `Content-Security-Policy` header'i oneriler:
  ```
  default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; media-src 'self';
  ```

## 11. Deployment

### 11.1 Build Command

```bash
npm run build
# Cikti: dist/ dizini (statik dosyalar)
```

### 11.2 Vite Config

```typescript
// vite.config.ts
import { defineConfig } from 'vite';

export default defineConfig({
  base: './',   // Goreceli yollar (alt dizine deploy icin)
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

### 11.3 Deploy Adimlari

```bash
# 1. Build
npm run build

# 2. dist/ dizinini sunucuya kopyala
scp -r dist/* user@server:/var/www/dinazor-oyunu/

# 3. Web sunucu yapilandirmasi (Nginx ornegi)
# location /dinazor-oyunu/ {
#   root /var/www;
#   try_files $uri $uri/ /dinazor-oyunu/index.html;
# }
```

## 12. Development Workflow

### 12.1 Local Setup

```bash
# 1. Bagimliliklari yukle
npm install

# 2. Gelistirme sunucusunu baslat
npm run dev
# Tarayicida http://localhost:3000 acilir

# 3. Tip kontrolu
npm run typecheck

# 4. Linting
npm run lint

# 5. Testleri calistir
npm run test

# 6. Production build
npm run build
```

### 12.2 Package.json Scripts

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

### 12.3 Code Standards

- **Dil**: TypeScript strict mode
- **Naming**: camelCase (degiskenler, fonksiyonlar), PascalCase (siniflar, tipler)
- **Formatter**: Prettier (2 space indent, single quotes, trailing comma)
- **Linter**: ESLint + @typescript-eslint
- **Commit convention**: Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`)

### 12.4 Git Workflow

- **Ana dal**: `main` (stabil, deploy edilebilir)
- **Feature branch**: `feature/<ozellik-adi>`
- **Bugfix branch**: `fix/<hata-aciklamasi>`
- **Merge strategy**: Squash merge (temiz tarihce)
