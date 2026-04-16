# DinazorKac -- Specification

> Chrome Dino tarzinda, yukaridan dusen meteorlardan kacan bir dinozor oyunu.

**Source:** Authored from discovery session on 2026-04-16.

## 1. Overview

### 1.1 What Is DinazorKac?

DinazorKac, tarayici tabanli bir 2D arcade oyunudur. Oyuncu, ekranin alt kisminda bulunan bir dinozoru klavye, mouse veya dokunmatik kontroller ile saga-sola hareket ettirerek yukaridan surekli dusen meteorlardan kacmaya calisir. Chrome'un cevrimdisi dinozor oyunundan esinlenmistir ancak farkli bir mekanik sunar: yatay hareket + dikey tehdit.

Oyun suresi ilerledikce zorluk artar -- meteorlar daha hizli ve daha sik duser. Oyuncu hayatta kaldigi surece puan toplar. Kalkan, yavaslatma ve cift puan gibi power-up'lar stratejik derinlik katar. Renkli pixel art gorsel stili ve tam ses destegi ile cilali bir oyun deneyimi sunar.

Proje acik kaynak olarak gelistirilecek, kendi sunucuya deploy edilecek ve hem masaustu hem mobil cihazlarda oynanabilir olacak sekilde tasarlanacaktir.

### 1.2 Target Audience

- **Casual oyuncular**: Kisa molalarda hizli bir oyun oynamak isteyen kullanicilar
- **Retro oyun severler**: Pixel art estetigi ve arcade mekanikleri seven kitle
- **Mobil kullanicilar**: Telefondan tarayici uzerinden oyun oynamak isteyenler
- **Gelistiriciler**: Acik kaynak bir Phaser.js projesini incelemek veya fork'lamak isteyenler

### 1.3 Key Differentiators

- **Ters dino mekanigi**: Chrome Dino'nun yatay kacis mekanigi yerine, dikey meteorlardan yatay kacis -- tanidik ama farkli bir deneyim
- **Tam mobil uyumluluk**: Dokunmatik kontroller ile mobilde sorunsuz oynanabilirlik
- **Power-up sistemi**: Klasik arcade oyunlarina stratejik derinlik katan toplanabilir guclendirmeler
- **Kademeli zorluk**: Oyun suresi ile orantili olarak artan meteor hizi ve yogunlugu
- **Retro pixel art**: Renkli ve canli pixel art gorselleri ile modern bir retro his

### 1.4 Competitive Landscape

| Feature             | DinazorKac         | Chrome Dino       | Asteroid Dodger (generic) |
|---------------------|--------------------|-------------------|---------------------------|
| Meteor kacis        | Yes                | No (engel atlama)  | Yes                       |
| Power-up sistemi    | Yes                | No                | Varies                    |
| Mobil dokunmatik    | Yes                | Limited           | Varies                    |
| Pixel art stili     | Renkli             | Siyah-beyaz       | Varies                    |
| Ses ve muzik        | Yes                | Minimal           | Varies                    |
| Zorluk seviyeleri   | Yes (3 seviye)     | No (kademeli)     | Varies                    |
| Acik kaynak         | Yes                | No                | Varies                    |

## 2. Core Concepts

| Concept          | Definition                                                                                         |
|------------------|----------------------------------------------------------------------------------------------------|
| Dinozor (Player) | Oyuncunun kontrol ettigi karakter; ekranin alt kisminda yatay olarak hareket eder                   |
| Meteor           | Ekranin ust kisminden rastgele konumlardan dusen tehlike nesnesi                                    |
| Power-Up         | Oyun alaninda beliren, toplandikta gecici avantaj saglayan ozel nesne                               |
| Skor             | Oyuncunun hayatta kaldigi sure ve topladigi bonuslara dayali sayisal deger                          |
| High Score       | Oyuncunun o tarayicida elde ettigi en yuksek skor (localStorage'da saklanir)                        |
| Zorluk Seviyesi  | Oyunun baslangic parametrelerini belirleyen on ayar: Kolay, Orta, Zor                               |
| Spawn Rate       | Meteorlarin birim zamanda olusturulma sikligi                                                       |
| Hit Box          | Dinozor ve meteorlarin carpisma algilama icin kullanilan sinir kutusu                                |
| Game Over        | Dinozorun bir meteora carpmasi sonucu oyunun sona ermesi                                            |
| Game Loop        | Oyunun surekli olarak guncellenmesi ve yeniden cizilmesi dongusu (Phaser Scene)                     |
| Shield           | Kalkan power-up'i: bir kez carpismadan korunma saglar                                               |
| Slow Motion      | Yavaslatma power-up'i: meteorlarin dusme hizini gecici olarak azaltir                               |
| Double Points    | Cift puan power-up'i: belirli bir sure boyunca kazanilan puanlari ikiyle carpar                      |

## 3. Functional Requirements

### 3.1 Core Gameplay

#### 3.1.1 Player Movement

**User Story:** As a player, I want to move the dinosaur left and right so that I can dodge falling meteors.

**Description:** Dinozor, ekranin alt kisminda sabit bir Y konumunda durur ve sadece yatay eksende hareket edebilir. Oyuncu, klavye ok tuslari (sol/sag), mouse hareketi veya dokunmatik surukle ile dinozoru kontrol eder. Dinozor ekranin disina cikamaz.

**Acceptance Criteria:**
- [ ] Sol/sag ok tuslari ile dinozor akici sekilde hareket eder
- [ ] A/D tuslari alternatif olarak desteklenir
- [ ] Mouse/dokunmatik ile dinozorun X konumu parmak/cursor pozisyonunu takip eder
- [ ] Dinozor ekranin sol ve sag sinirlarini asamaz
- [ ] Hareket hizi sabit ve tutulan tusa/dokunusa bagli olarak sureklidir
- [ ] Masaustu ve mobil cihazlarda esit derecede duyarli kontrol

**Edge Cases:**
- Birden fazla kontrol yontemi ayni anda kullanilirsa, son girdi onceliklidir
- Ekran boyutu degistiginde dinozor ekran icinde kalir

**Constraints:**
- Dinozor genisligi oyun alaninin %8-12'sini kaplar
- Hareket hizi: 300-500 piksel/saniye (ayarlanabilir)

#### 3.1.2 Meteor Spawning & Falling

**User Story:** As a player, I want meteors to fall from the top so that I have obstacles to dodge.

**Description:** Meteorlar ekranin ust kenarinin uzerinde rastgele X konumlarinda olusturulur ve asagi dogru duser. Zamanla meteor olusma sikligi ve dusme hizi artar. Meteorlarin boyutlari kucuk, orta ve buyuk olarak cesitlilik gosterir.

**Acceptance Criteria:**
- [ ] Meteorlar ekranin ust kenarinin uzerinde rastgele X pozisyonlarinda olusur
- [ ] Meteorlar sabit veya hafif degisen hizda asagi duser
- [ ] Farkli boyutlarda meteorlar olusur (kucuk: 24px, orta: 40px, buyuk: 56px)
- [ ] Meteor spawn rate zamanla kademeli olarak artar
- [ ] Meteor dusme hizi zamanla kademeli olarak artar
- [ ] Ekranin altina ulasan meteorlar yok edilir ve bellekten temizlenir
- [ ] Ayni anda maksimum 30 meteor ekranda bulunabilir

**Edge Cases:**
- Tum meteorlar ekranin bir tarafinda olusursa, oyuncu kacamaz hale gelebilir -- minimum yatay dagilim garanti edilmeli
- Performans dususu durumunda spawn rate gecici olarak azaltilir

**Constraints:**
- Baslangic spawn rate: zorluga bagli (Kolay: 800ms, Orta: 600ms, Zor: 400ms)
- Baslangic dusme hizi: zorluga bagli (Kolay: 150px/s, Orta: 250px/s, Zor: 350px/s)
- Maksimum dusme hizi: 600px/s
- Hiz artis orani: her 10 saniyede %5

#### 3.1.3 Collision Detection

**User Story:** As a player, I want accurate collision detection so that gameplay feels fair.

**Description:** Dinozor ile meteor arasinda carpisma algilandiginda oyun sona erer (kalkan aktif degilse). Carpisma kontrolu sprite hitbox'larina dayalidir ve gorselden biraz kucuk ayarlanarak "adil" bir his saglanir.

**Acceptance Criteria:**
- [ ] Dinozor-meteor temasinda carpismaya algilanir
- [ ] Hitbox'lar sprite boyutunun %80'i olarak ayarlanir (forgiveness margin)
- [ ] Kalkan aktifken carpisma dinozoru yok etmez, kalkan tuketilir
- [ ] Carpisma aninda gorsel geri bildirim (parlama/titreme efekti) verilir
- [ ] Carpisma aninda ses efekti calinir

**Edge Cases:**
- Ayni frame'de birden fazla meteor carpisirsa, yalnizca bir Game Over tetiklenir
- Kalkan + carpisma esanli olursa, kalkan onceliklidir

#### 3.1.4 Scoring System

**User Story:** As a player, I want to earn points while surviving so that I can track my performance.

**Description:** Oyuncu hayatta kaldigi her saniye icin puan kazanir. Puan, ekranin ust kosesinde surekli guncellenir. Cift puan power-up'i aktifken kazanilan puan iki katina cikar.

**Acceptance Criteria:**
- [ ] Her 100ms'de 1 puan eklenir (saniyede 10 puan)
- [ ] Cift puan power-up'i aktifken puan kazanim hizi iki katina cikar
- [ ] Anlık skor ekranin sag ust kosesinde goruntulenir
- [ ] Skor formatı: "Skor: 1,234" (binlik ayiraci ile)
- [ ] Game Over ekraninda son skor ve en yuksek skor yan yana gosterilir

**Edge Cases:**
- Skor integer overflow'a ulasamaz (pratikte mumkun degil -- max ~86,400,000 / gun)

### 3.2 Menu & UI

#### 3.2.1 Main Menu

**User Story:** As a player, I want a main menu so that I can start the game and adjust settings.

**Description:** Oyun acildiginda ana menu ekrani goruntulenir. Menu, oyun basligi, baslat butonu, zorluk secimi, en yuksek skor gosterimi ve ses ayarlari icerir.

**Acceptance Criteria:**
- [ ] "DinazorKac" basligi pixel art fontla goruntulenir
- [ ] "Oyna" butonu goruntulenir ve tiklanabilir/dokunulabilir
- [ ] Zorluk secimi (Kolay / Orta / Zor) goruntulenir, varsayilan: Orta
- [ ] En yuksek skor goruntulenir
- [ ] Ses acma/kapama butonu goruntulenir
- [ ] Arka plan animasyonu veya dekoratif gorseller menude bulunur

#### 3.2.2 HUD (Heads-Up Display)

**User Story:** As a player, I want to see my score and active power-ups during gameplay.

**Description:** Oyun sirasinda ekranda skor, aktif power-up'lar ve sure gostergeleri goruntulenir.

**Acceptance Criteria:**
- [ ] Skor sag ust kosede goruntulenir
- [ ] Aktif power-up ikonlari ve kalan sureleri sol ust kosede goruntulenir
- [ ] HUD elemanlari oyun alanini engellemez
- [ ] HUD, mobil ve masaustunde okunabilir boyuttadir

#### 3.2.3 Game Over Screen

**User Story:** As a player, I want to see my results after dying so that I can choose to retry.

**Description:** Dinozor bir meteora carptiginda Game Over ekrani goruntulenir. Son skor, en yuksek skor (yeni rekor ise vurgulu), tekrar oyna ve ana menuye don butonlari gosterilir.

**Acceptance Criteria:**
- [ ] "Game Over" yazisi animasyonlu sekilde goruntulenir
- [ ] Son skor goruntulenir
- [ ] En yuksek skor goruntulenir
- [ ] Yeni rekor kirilmissa ozel animasyon/efekt gosterilir
- [ ] "Tekrar Oyna" butonu goruntulenir
- [ ] "Ana Menu" butonu goruntulenir
- [ ] Herhangi bir tusa basarak veya dokunarak hizli tekrar baslama desteklenir

#### 3.2.4 Pause Menu

**User Story:** As a player, I want to pause the game so that I can take a break.

**Description:** Oyun sirasinda Escape tusu veya ekrandaki duraklat butonuna basildiginda oyun duraklar.

**Acceptance Criteria:**
- [ ] Escape tusu veya duraklat butonuyla oyun duraklar
- [ ] Duraklama sirasinda oyun dongusu durur
- [ ] "Devam Et", "Yeniden Basla", "Ana Menu" butonlari goruntulenir
- [ ] Mobilde ekrana dokunma ile duraklama tetiklenmez (ozel buton gerekir)

### 3.3 Power-Up System

#### 3.3.1 Power-Up Spawning

**User Story:** As a player, I want power-ups to appear during gameplay so that I can gain advantages.

**Description:** Power-up'lar belirli araliklarla rastgele konumlarda olusur ve yavas yavas asagi duser. Oyuncu dinozoruyla temas ederek toplar.

**Acceptance Criteria:**
- [ ] Power-up'lar her 15-25 saniyede bir rastgele olusur
- [ ] Power-up'lar meteorlardan yavas duser
- [ ] Her power-up tipi farkli renk ve ikonla ayirt edilir
- [ ] Toplandiginda gorsel ve ses efekti olusur
- [ ] Toplanmayan power-up'lar ekranin altina ulastiginda yok edilir

#### 3.3.2 Shield Power-Up

**Description:** Dinozorun etrafinda gorsel bir kalkan olusturur. Bir carpismayi emer ve kaybolur.

**Acceptance Criteria:**
- [ ] Toplandikta dinozorun etrafinda mavi parlayan kalkan gorseli belirir
- [ ] Bir meteor carpismasi emilir ve dinozor hayatta kalir
- [ ] Kalkan kullanildiginda kirilan kalkan animasyonu oynar
- [ ] Ayni anda yalnizca bir kalkan aktif olabilir
- [ ] Kalkan suresi: sinirsiz (sadece carpisma ile tuketilir)

#### 3.3.3 Slow Motion Power-Up

**Description:** Tum meteorlarin dusme hizini gecici olarak %50 yavaslatir.

**Acceptance Criteria:**
- [ ] Toplandikta tum meteorlarin hizi %50 azalir
- [ ] Etki suresi: 5 saniye
- [ ] Aktif oldugunda ekranda gorsel gosterge (mavi ton filtresi veya partikuller) belirir
- [ ] Sure bitiminde hiz normale doner (ani degil, 1 saniyede kademeli)
- [ ] Birden fazla yavaslatma ust uste yigilmaz -- sureyi sifirlar

#### 3.3.4 Double Points Power-Up

**Description:** Belirli bir sure boyunca kazanilan puanlar iki katina cikar.

**Acceptance Criteria:**
- [ ] Toplandikta skor carpani x2 olur
- [ ] Etki suresi: 8 saniye
- [ ] Aktif oldugunda skor gostergesi altin rengi ile vurgulanir
- [ ] Sure bitiminde normal puan kazanima donulur
- [ ] Birden fazla cift puan ust uste yigilmaz -- sureyi sifirlar

### 3.4 Difficulty System

#### 3.4.1 Difficulty Levels

**User Story:** As a player, I want to choose a difficulty level so that I can match the challenge to my skill.

**Description:** Oyun baslangicinda uc zorluk seviyesi sunulur. Her seviye, meteor hizi, spawn rate ve power-up sikligini belirler.

**Acceptance Criteria:**
- [ ] Uc seviye sunulur: Kolay, Orta, Zor
- [ ] Secilen seviye ana menude vurgulanir
- [ ] Secim localStorage'a kaydedilir (son tercih hatirlenir)

**Difficulty Parameters:**

| Parameter              | Kolay   | Orta    | Zor     |
|------------------------|---------|---------|---------|
| Baslangic meteor hizi  | 150px/s | 250px/s | 350px/s |
| Baslangic spawn rate   | 800ms   | 600ms   | 400ms   |
| Hiz artis orani        | %3/10s  | %5/10s  | %7/10s  |
| Power-up sikligi       | 15s     | 20s     | 25s     |
| Dinozor hareket hizi   | 400px/s | 350px/s | 300px/s |

#### 3.4.2 Progressive Difficulty

**Description:** Secilen zorluk seviyesinden bagimsiz olarak, oyun suresi ilerledikce meteorlar hizlanir ve siklaslir.

**Acceptance Criteria:**
- [ ] Her 10 saniyede meteor dusme hizi artar (seviyeye bagli oran)
- [ ] Her 10 saniyede meteor spawn rate azalir (daha sik olusma)
- [ ] Artis oranlari zorluk seviyesine gore degisir
- [ ] Maksimum hiz ve minimum spawn rate sinirlari vardir (oynanamaz hale gelmemeli)

### 3.5 High Score System

#### 3.5.1 Local High Score

**User Story:** As a player, I want my best score saved so that I can try to beat it.

**Description:** Oyuncunun her zorluk seviyesi icin en yuksek skoru localStorage'da saklanir.

**Acceptance Criteria:**
- [ ] Her zorluk seviyesi icin ayri high score kaydedilir
- [ ] Yeni rekor kirildiginda Game Over ekraninda "Yeni Rekor!" animasyonu gosterilir
- [ ] High score ana menude goruntulenir (secili zorlugun skoru)
- [ ] localStorage'a JSON formatinda kaydedilir
- [ ] localStorage erisimi yoksa oyun calismaya devam eder (skor saklanmaz)

### 3.6 Audio System

#### 3.6.1 Background Music

**Description:** Oyun sirasinda dongusel arka plan muzigi calar.

**Acceptance Criteria:**
- [ ] Ana menude ayri bir muzik parcasi calar
- [ ] Oyun sirasinda ayri bir muzik parcasi calar
- [ ] Muzik dongusel olarak tekrarlanir
- [ ] Ses seviyesi ayarlanabilir veya tamamen kapatilabilir
- [ ] Muzik/SFX tercihi localStorage'da saklanir
- [ ] Mobilde tarayici otomatik calistirma kisitlamasina uyumlu (ilk etkilesimden sonra baslar)

#### 3.6.2 Sound Effects

**Description:** Oyun ici olaylarda ses efektleri calinir.

**Acceptance Criteria:**
- [ ] Meteor carpisma sesi
- [ ] Power-up toplama sesi
- [ ] Kalkan kirilma sesi
- [ ] Game Over sesi
- [ ] Yeni rekor sesi
- [ ] Buton tiklama sesi
- [ ] SFX bagimsiz olarak acilip kapatilabilir

## 4. Architecture Overview

### 4.1 System Components

| Component      | Description                                                                      |
|----------------|----------------------------------------------------------------------------------|
| Game Engine    | Phaser.js 3.x -- oyun dongusu, render, fizik ve girdi yonetimi                   |
| Scene Manager  | Phaser scene sistemi -- BootScene, MenuScene, GameScene, GameOverScene            |
| Entity System  | Player, Meteor ve PowerUp sprite siniflari                                       |
| HUD Manager    | Skor, power-up gostergeleri ve duraklama kontrollerini yoneten UI katmani        |
| Audio Manager  | Muzik ve ses efektlerini yoneten merkezi ses kontrolcu                            |
| Score Manager  | Skor hesaplama, high score kayit ve localStorage erisimini yoneten modul         |
| Config Manager | Zorluk parametreleri ve oyun sabitleri icin merkezi yapilandirma                 |
| Input Handler  | Klavye, mouse ve dokunmatik girdileri birlestiren girdi katmani                   |

### 4.2 Component Interactions

```
[Input Handler] --> [Player Entity] --> [Game Scene]
                                            |
                    [Meteor Spawner] ------->|
                    [PowerUp Spawner] ------>|
                                            |
                    [Collision Manager] <----|
                         |
            +-----------+-----------+
            |           |           |
      [HUD Manager] [Audio Manager] [Score Manager]
                                        |
                                  [localStorage]
```

- Input Handler, kullanici girdilerini alir ve Player Entity'ye iletir
- Game Scene, tum entity'leri olusturur, gunceller ve yonetir
- Collision Manager, carpismalari algilar ve ilgili manager'lari bilgilendirir
- Score Manager, skor hesaplar ve localStorage ile eslestir
- Audio Manager, olay tabanli ses calma islemlerini yonetir
- HUD Manager, skor ve power-up durumunu gorsel olarak yansitir

### 4.3 External Integrations

Bu proje disariya bagimliligi minimumdur:

| Integration    | Purpose                                  | Fallback                                  |
|----------------|------------------------------------------|-------------------------------------------|
| localStorage   | High score ve tercih saklama             | Oyun calisir, veriler session boyunca tutulur |
| Web Audio API  | Ses oynatma (Phaser tarafindan yonetilir) | Sessiz mod                                |

## 5. Data Model

### 5.1 Core Entities

#### GameState (Runtime)

| Field           | Type        | Description                            |
|-----------------|-------------|----------------------------------------|
| score           | number      | Anlik skor                             |
| isGameOver      | boolean     | Oyun bitti mi?                         |
| isPaused        | boolean     | Oyun duraksatildi mi?                  |
| difficulty      | enum        | Kolay / Orta / Zor                     |
| elapsedTime     | number      | Gecen sure (ms)                        |
| activePowerUps  | PowerUp[]   | Aktif power-up listesi                 |
| currentSpeed    | number      | Mevcut meteor dusme hizi               |
| currentSpawnRate| number      | Mevcut meteor olusma sikligi           |

#### Player (Runtime)

| Field      | Type    | Description                      |
|------------|---------|----------------------------------|
| x          | number  | Yatay konum                      |
| y          | number  | Dikey konum (sabit)              |
| width      | number  | Sprite genisligi                 |
| height     | number  | Sprite yuksekligi                |
| hasShield  | boolean | Kalkan aktif mi?                 |
| speed      | number  | Hareket hizi (px/s)              |

#### Meteor (Runtime)

| Field    | Type   | Description                |
|----------|--------|----------------------------|
| x        | number | Yatay konum                |
| y        | number | Dikey konum                |
| size     | enum   | small / medium / large     |
| speed    | number | Dusme hizi (px/s)          |
| rotation | number | Donme acisi (gorsel)       |

#### PowerUp (Runtime)

| Field     | Type   | Description                         |
|-----------|--------|-------------------------------------|
| x         | number | Yatay konum                         |
| y         | number | Dikey konum                         |
| type      | enum   | shield / slowMotion / doublePoints  |
| duration  | number | Etki suresi (ms), shield icin null  |
| speed     | number | Dusme hizi (px/s)                   |

### 5.2 Persisted Data (localStorage)

```json
{
  "dinazorKac": {
    "highScores": {
      "easy": 0,
      "medium": 0,
      "hard": 0
    },
    "settings": {
      "difficulty": "medium",
      "musicEnabled": true,
      "sfxEnabled": true
    }
  }
}
```

### 5.3 Data Lifecycle

- **GameState**: Oyun basladiginda olusturulur, Game Over'da sifirlanir. Kalici saklanmaz.
- **High Score**: Game Over'da mevcut skor ile karsilastirilir. Daha yuksekse localStorage'a yazilir.
- **Settings**: Kullanici degistirdiginde aninda localStorage'a yazilir.
- **Silme**: localStorage temizleme yalnizca tarayici ayarlarindan yapilir. Oyun icinde sifirlama secenegi yoktur.

## 7. User Interface

### 7.1 Interface Type

Web UI -- HTML5 Canvas uzerinde Phaser.js ile render edilen tam ekran oyun arayuzu.

### 7.2 Key Screens

#### Boot Screen
- **Purpose**: Asset'leri yuklemek
- **Elements**: Yukleme cubugu, "Yukleniyor..." yazisi
- **Actions**: Otomatik gecis (yukleme tamamlaninca MenuScene'e)

#### Main Menu
- **Purpose**: Oyunu baslatmak ve ayar yapmak
- **Elements**: Baslik, Oyna butonu, zorluk secici, high score, ses butonu
- **Actions**: Oyna'ya bas -> GameScene'e gecis
- **Navigation**: GameScene'e ve ayarlara gecis

#### Game Screen
- **Purpose**: Oyun oynama alani
- **Elements**: Dinozor, meteorlar, power-up'lar, HUD (skor, aktif power-up'lar, duraklat butonu)
- **Actions**: Hareket, power-up toplama, duraklat
- **Navigation**: GameOverScene'e (olum) veya PauseMenu'ye (duraklat)

#### Pause Overlay
- **Purpose**: Oyunu duraklatmak
- **Elements**: Karartilmis arka plan, Devam Et / Yeniden Basla / Ana Menu butonlari
- **Actions**: Devam, yeniden baslat, menuye don
- **Navigation**: GameScene'e geri veya MenuScene'e

#### Game Over Screen
- **Purpose**: Sonuclari gostermek ve tekrar oynamayi tesvik etmek
- **Elements**: Son skor, en yuksek skor, yeni rekor animasyonu, Tekrar Oyna / Ana Menu butonlari
- **Actions**: Tekrar oyna, menuye don
- **Navigation**: GameScene'e veya MenuScene'e

### 7.3 Responsive Requirements

- **Minimum genislik**: 320px (kucuk telefonlar)
- **Maksimum genislik**: Sinirsiz (masaustu tam ekran)
- **Aspect ratio**: Sabit degil -- oyun alani pencere boyutuna uyum saglar
- **Canvas olcekleme**: Phaser'in Scale Manager'i ile `RESIZE` veya `FIT` modu
- **Dokunmatik hedefler**: Minimum 44x44px (Apple HIG standardi)
- **Font olcekleme**: Canvas boyutuna oranli
- **Breakpoint'ler**: 
  - Mobil: < 768px (dokunmatik kontroller goruntulenir)
  - Masaustu: >= 768px (klavye/mouse kontrolleri oncelikli)

## 9. Deployment Model

### 9.1 Target Environments

- **Production**: Kullanicinin kendi sunucusu (web server)
- **Development**: Lokal gelistirme ortami (Vite dev server)

### 9.2 Distribution Method

Statik dosyalar olarak sunulur. Build ciktisi (HTML + JS + assets) herhangi bir web sunucuya kopyalanarak deploy edilir. Docker opsiyonel olarak desteklenebilir.

### 9.3 Configuration

- Build-time: `vite.config.js` ile asset yollari ve build ayarlari
- Runtime: Oyun icindeki Config modulu ile oyun parametreleri (kod icinde sabit degerler)

### 9.4 System Requirements

- **Sunucu**: Herhangi bir statik dosya sunucusu (Nginx, Apache, Caddy, vb.)
- **Istemci tarayici**: Chrome 80+, Firefox 75+, Safari 13+, Edge 80+
- **Minimum cihaz**: 2GB RAM, herhangi bir modern GPU (entegre dahil)

## 10. Performance Requirements

### 10.1 Frame Rate Targets

- **Hedef FPS**: 60 FPS sabit
- **Minimum kabul edilebilir FPS**: 30 FPS (dusuk performansli cihazlarda)
- **Frame drop toleransi**: Art arda 5 frame'den fazla dusus olmamalI

### 10.2 Asset Loading

- **Ilk yukleme suresi**: < 3 saniye (ortalama baglanti hizi)
- **Toplam asset boyutu**: < 5MB (sprite sheet + ses dosyalari dahil)
- **Sprite sheet optimizasyonu**: Tum gorseller tek atlas'ta birlestirilmeli

### 10.3 Memory Usage

- **Maksimum heap kullanimi**: < 100MB
- **Object pooling**: Meteorlar ve power-up'lar icin nesne havuzu kullanilmali (GC basinc azaltma)
- **Yok edilen sprite'lar**: Ekran disina cikan nesneler aninda destroy edilmeli

## 11. Constraints & Non-Goals

### 11.1 Technical Constraints

- Phaser.js 3.x framework'une bagimlidir
- HTML5 Canvas/WebGL destekleyen modern tarayici gerektirir
- Ses, ilk kullanici etkilesiminden sonra baslatilabilir (tarayici politikasi)
- localStorage kapasitesi: ~5MB (yeterli)

### 11.2 Non-Goals

- **Cok oyunculu mod**: Online multiplayer bu kapsamda yoktur. Gelecekte dusunulebilir.
- **Backend / sunucu tarafli mantik**: Tum oyun istemci tarafinda calisir. Global leaderboard yoktur.
- **Hesap sistemi**: Kullanici kayit/giris yoktur. Tum veriler yereldir.
- **Oyun ici satin alma**: Hicbir monetizasyon mekanizmasi yoktur.
- **Seviye tasarimcisi**: Oyuncular ozel seviye olusturamaz.
- **Fizik motoru**: Gercekci fizik simulasyonu yoktur. Basit kinematik hareket yeterlidir.
- **Yapay zeka**: Dusman AI'i yoktur. Meteorlar rastgele duser.
- **Offline PWA**: Service Worker ile cevrimdisi destek bu kapsamda yoktur.
- **Coklu dil destegi (i18n)**: Arayuz dili yalnizca Turkcedir.
- **Gamepad destegi**: Yalnizca klavye, mouse ve dokunmatik desteklenir.

### 11.3 Assumptions

- Hedef kullanicilar modern tarayiciya sahiptir (IE destegi yoktur)
- localStorage tum hedef tarayicilarda kullanilabilirdir
- Pixel art asset'leri gelistirme sirasinda olusturulacaktir (harici tasarimci gerektirmez)
- Ses dosyalari telif hakki olmayan kaynaklardan temin edilecektir

### 11.4 Open Questions

- **[TBD: Tam ekran modu]**: Oyun tam ekran modunu desteklemeli mi? Mobilde faydali olabilir.
- **[TBD: Sosyal paylasim]**: Game Over'da skor paylasma butonu olsun mu?

## 12. Future Considerations

- **v1.1**: Global leaderboard -- basit bir backend ile online skor tablosu
- **v1.1**: Ek power-up turleri -- magnet (power-up'lari ceker), bomba (tum meteorlari temizler)
- **v1.2**: Tema/skin sistemi -- farkli dinozor ve arka plan temalari
- **v1.2**: PWA destegi -- Service Worker ile cevrimdisi oynanabilirlik
- **v2.0**: Cok oyunculu mod -- ayni ekranda veya online rakiplerle yarisma
- **v2.0**: Gamepad destegi
