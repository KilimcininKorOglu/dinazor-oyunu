# DinazorKac -- Branding Guide

## 1. Name & Identity

### 1.1 Project Name
- **Name**: DinazorKac
- **Pronunciation**: Di-na-zor-kaç
- **Etymology**: "Dinazor" (Turkce dinozor) + "Kac" (kacmak fiilinden) -- Dinazor Kac! yani "Dinozor, Kac!"
- **In code**: `dinazor-kac`
- **In prose**: DinazorKac (tek kelime, PascalCase)

### 1.2 Tagline
- **Primary**: Meteorlardan kac, hayatta kal!
- **Technical**: Phaser.js ile yapilmis 2D arcade meteor kacis oyunu
- **Marketing**: Dinozorunu kontrol et, meteorlardan kac, rekoru kir!

### 1.3 Elevator Pitch
DinazorKac, Chrome'un cevrimdisi dino oyunundan esinlenmis bir tarayici tabanli arcade oyunudur. Oyuncu, yukaridan surekli dusen meteorlardan kacan bir dinozoru klavye, mouse veya dokunmatik ile kontrol eder. Artan zorluk, toplanabilir guclendirmeler ve renkli pixel art estetigi ile bagimlilik yapan bir deneyim sunar. Hem masaustunde hem mobilde sorunsuz calisir.

## 2. Logo

### 2.1 Concept
Kucuk, sevimli bir pixel art dinozor silueti -- yesil tonlarda, arkasinda turuncu bir meteor izi ile. Dinozor saga dogru kosuyormus gibi dinamik bir poz. Retro arcade hissiyatini tasir.

### 2.2 Specifications
- **Primary mark**: Pixel art dinozor + "DinazorKac" yazisi yan yana
- **Icon mark**: Sadece dinozor silueti (favicon, app ikonu icin)
- **Wordmark**: "DinazorKac" pixel fontta
- **Minimum size**: 32x32px (icon), 120x40px (wordmark)
- **Clear space**: Logo etrafinda logo yuksekliginin %25'i kadar bosluk

### 2.3 AI Generation Prompt
```
Pixel art style logo of a cute green dinosaur (T-Rex) running to the right,
8-bit retro game aesthetic, flat colors, no gradients. The dinosaur is small
and cartoonish with short arms and a big head. Behind the dinosaur, a trail of
orange/red meteor fragments. Dark navy blue (#1a1a2e) background. Square format
(1:1 ratio). Clean pixel edges, no anti-aliasing. Colors limited to: green
(#4ade80), dark green (#166534), orange (#f97316), dark orange (#c2410c), and
background navy (#1a1a2e). No text in the image. Style reference: classic
NES/SNES sprite art.
```

## 3. Color Palette

### 3.1 Brand Colors

| Role      | Name         | Hex     | RGB             | Usage                                    |
|-----------|-------------|---------|-----------------|------------------------------------------|
| Primary   | Dino Green  | #4ade80 | rgb(74,222,128) | Dinozor, ana butonlar, aktif durumlar     |
| Secondary | Meteor Fire | #f97316 | rgb(249,115,22) | Meteorlar, vurgu efektleri, tehlike       |
| Accent    | Power Gold  | #fbbf24 | rgb(251,191,36) | Puan gostergesi, cift puan, basarilar     |

### 3.2 Game Colors

| Role            | Name          | Hex     | Usage                                        |
|-----------------|--------------|---------|----------------------------------------------|
| Background      | Deep Navy    | #1a1a2e | Ana oyun arka plan, gece gokyuzu             |
| Surface         | Dark Indigo  | #16213e | Menu panelleri, HUD arka plani               |
| Text Primary    | Pure White   | #ffffff | Basliklar, skor, menu yazilari               |
| Text Secondary  | Soft Gray    | #94a3b8 | Alt yazilar, ipucu metinleri                 |
| Border          | Slate        | #334155 | Panel kenarliklari, ayiricilar               |
| Shield Blue     | Cyan Glow    | #22d3ee | Kalkan power-up, kalkan gorseli              |
| Slow Green      | Emerald      | #10b981 | Yavaslatma power-up                          |
| Star Field      | Dim Star     | #475569 | Arka plan yildizlari                         |

### 3.3 Semantic Colors

| Role    | Hex     | Usage                              |
|---------|---------|-------------------------------------|
| Success | #4ade80 | Yeni rekor, basarili islem          |
| Error   | #ef4444 | Carpisma, game over, hata          |
| Warning | #f59e0b | Kalkan kirilmasi, yaklasan tehlike  |
| Info    | #3b82f6 | Bilgi mesajlari, ipuclari          |

### 3.4 Phaser Color Constants

```typescript
// src/config/Colors.ts
export const Colors = {
  // Brand
  dinoGreen:    0x4ade80,
  meteorFire:   0xf97316,
  powerGold:    0xfbbf24,

  // Background
  deepNavy:     0x1a1a2e,
  darkIndigo:   0x16213e,

  // UI
  white:        0xffffff,
  softGray:     0x94a3b8,
  slate:        0x334155,

  // Power-ups
  shieldCyan:   0x22d3ee,
  slowEmerald:  0x10b981,

  // Semantic
  success:      0x4ade80,
  error:        0xef4444,
  warning:      0xf59e0b,
} as const;
```

## 4. Typography

### 4.1 In-Game Font

| Role      | Font              | Style           | Fallback              |
|-----------|------------------|-----------------|-----------------------|
| All text  | Pixel Bitmap Font | 8x8 pixel grid | Phaser default bitmap |

Oyun ici tum metinler Phaser BitmapText ile render edilir. Web font bagimliligi yoktur.

**Pixel font ozellikleri:**
- Monospace, 8x8 piksel karakter matrisi
- Turkce karakter destegi (c, g, i, I, o, s, u) -- onemli!
- Beyaz ana renk, Phaser tint ile renklendirilir

### 4.2 Web/README Font

| Role     | Font                  | Weights | Fallback       |
|----------|-----------------------|---------|----------------|
| Headings | Press Start 2P (Google) | 400   | monospace      |
| Body     | Inter                  | 400,600 | system-ui      |

README ve web sayfasi (varsa) icin kullanilir. Oyun icinde kullanilmaz.

## 5. Voice & Tone

### 5.1 Personality
- **Eglenceli**: Oyun bir eglence urunudur -- dil samimi ve enerjik olsun. "Meteorlardan kac, hayatta kal!"
- **Dogrudan**: Gereksiz aciklama yok. "Oyna" butonu, "Tekrar Oyna", "Ana Menu" -- kisa ve net.
- **Motive edici**: Oyuncuyu tekrar oynamaya tesvik et. "Yeni Rekor!" yazisi, skor karsilastirmasi.
- **Turkce**: Tum oyun ici metinler Turkcedir. Teknik terimler hariç.

### 5.2 In-Game Text Rules
- Buton metinleri: 1-2 kelime, fiil veya isim ("Oyna", "Devam Et", "Ana Menu")
- Bildirimler: Kisa ve vurgulu ("Yeni Rekor!", "Kalkan Kirildi!", "x2 Puan!")
- Zorluk isimleri: "Kolay", "Orta", "Zor"
- Skor etiketleri: "Skor:", "En Yuksek:"

### 5.3 Vocabulary

| Prefer          | Avoid               |
|-----------------|----------------------|
| Oyna            | Basla / Play         |
| Tekrar Oyna     | Restart              |
| Ana Menu        | Geri / Back          |
| Devam Et        | Resume               |
| Skor            | Puan / Score         |
| En Yuksek       | High Score / Rekor   |
| Yeni Rekor!     | New High Score!      |
| Game Over       | Oyun Bitti (Game Over evrensel, kalabilir) |

## 6. Visual Language

### 6.1 Pixel Art Style Rules
- **Cozunurluk**: Tum sprite'lar 16x16 veya 32x32 temel izgarada ciziilir
- **Palette siniri**: Sprite basina maksimum 8 renk
- **Outline**: 1px koyu outline (siyah veya koyu versiyon) tum sprite'larda
- **Anti-alias yok**: Sert piksel kenarlari, yamusamis kenarlar yok
- **Golge**: Basit 1-2 tonlu golgelendirme, gradient yok
- **Animasyon**: 2-4 frame ile basit dongusel animasyonlar

### 6.2 UI Style
- **Paneller**: Koyu arka plan (#16213e), 1px border (#334155), 4px kose yuvarlama (piksel hissiyatinda)
- **Butonlar**: Dolu arka plan (marka rengi), hover'da parlama efekti, tikta basin efekti
- **Metin golge**: Oyun ici metinlerde 1px siyah golge (okunabilirlik icin)

### 6.3 Spacing
- **Base unit**: 8px (piksel izgaraya uyumlu)
- **Scale**: 8, 16, 24, 32, 48, 64

## 7. Assets Checklist

| Asset                | Format      | Size          | Status |
|----------------------|------------|---------------|--------|
| Logo (full)          | PNG        | 512x512       | [TBD]  |
| Icon (favicon)       | PNG + ICO  | 32x32, 16x16  | [TBD]  |
| Dino sprite sheet    | PNG        | 128x32        | [TBD]  |
| Meteor sprites       | PNG        | 64x192        | [TBD]  |
| Power-up icons       | PNG        | 48x16         | [TBD]  |
| Background           | PNG        | 480x720       | [TBD]  |
| UI atlas             | PNG        | 256x256       | [TBD]  |
| Pixel bitmap font    | PNG + XML  | 128x128       | [TBD]  |
| OG Image (sosyal)    | PNG        | 1200x630      | [TBD]  |
