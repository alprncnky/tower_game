# Tower Game — Proje Yapısı ve AI Geliştirme Rehberi

Bu belge, **Tower Building Game** (Tower Bloxx tarzı kule inşa oyunu) projesinin teknik mimarisini, kullanılan yöntemleri ve dosya sorumluluklarını AI ajanları için özetler. Kod değişikliği yapmadan önce bu belgeyi okuyun.

---

## 1. Proje Özeti

| Alan | Değer |
|------|-------|
| Tür | 2D Canvas tabanlı mobil uyumlu mini oyun |
| Orijin | [iamkun/tower_game](https://github.com/iamkun/tower_game) (MIT) |
| Oyun mantığı | Vinçten sallanan blokları dokunarak bırak; üst üste istifle |
| Can sistemi | 3 kalp; blok düşerse −1 can; 3 hata = oyun biter |
| Skor | Success: +25, Perfect: +50 (+ ardışık perfect bonusu) |

**İki katmanlı mimari:**

1. **HTML/CSS kabuk** (`index.html`) — yükleme ekranı, landing, game-over modal, responsive layout
2. **Canvas oyun motoru** (`src/` → `dist/main.js`) — tüm oyun fiziği, çizim, ses

Kabuk ve oyun `window.TowerGame(option)` factory fonksiyonu ve callback hook'ları ile bağlanır.

---

## 2. Teknoloji Yığını

### Çalışma zamanı

| Araç | Rol |
|------|-----|
| **HTML5 Canvas** | Oyun render katmanı |
| **cooljs** (`^1.0.2`) | Hafif oyun motoru: `Engine`, `Instance`, asset yükleme, game loop, time movement |
| **Zepto.js** (`assets/zepto-1.1.6.min.js`) | DOM manipülasyonu, event binding (jQuery benzeri, hafif) |
| **Vanilla JS (ES6 modules)** | Oyun mantığı kaynak kodu |

### Build & geliştirme

| Araç | Rol |
|------|-----|
| **Webpack 4** | `src/` → tek bundle `dist/main.js` (cooljs dahil bundle edilir) |
| **Babel 7** (`@babel/preset-env`) | ES6+ → hedef: iOS ≥9, Android ≥4 |
| **Express 4** | Statik dosya sunucusu (`index.js`, port 8082) |
| **opn** | `npm start` sonrası tarayıcıyı otomatik açar |

### Komutlar

```bash
npm install          # bağımlılıkları kur
npm run build        # webpack production build
npm start            # build + express sunucu (http://localhost:8082)
```

**Not:** Build komutu `NODE_OPTIONS=--openssl-legacy-provider` kullanır (Node.js OpenSSL 3 uyumluluğu için). Webpack config dosyası yok; Webpack 4 varsayılan entry: `src/index.js`.

---

## 3. Dizin Yapısı

```
tower_game/
├── index.html          # UI kabuk, oyun başlatma, option hook'ları
├── index.js            # Express dev sunucusu
├── package.json
├── .babelrc            # Babel hedef tarayıcılar
├── src/                # Oyun kaynak kodu (ES6 modüller)
│   ├── index.js        # TowerGame factory — motor kurulumu, asset kaydı
│   ├── constant.js     # String sabitler (state key'leri, movement ID'leri)
│   ├── utils.js        # Skor, zorluk, dokunma, çizim yardımcıları
│   ├── block.js        # Blok FSM, çarpışma, düşme fiziği
│   ├── hook.js         # Vinç/kanca animasyonu
│   ├── line.js         # Üstteki referans çizgisi (collision surface)
│   ├── background.js   # Gradient + arka plan görseli
│   ├── cloud.js        # Dekoratif bulut/taş parallax
│   ├── flight.js       # Uçan objeler (belirli katlarda spawn)
│   ├── tutorial.js     # İlk dokunuş tutorial overlay
│   └── animateFuncs.js # startAnimate / endAnimate (HUD + yeni blok spawn)
├── dist/
│   └── main.js         # Webpack çıktısı (commit'lenmiş, doğrudan HTML'den yüklenir)
├── assets/             # Görseller, sesler, font, Zepto (build'e dahil değil, statik)
└── docs/
    └── PROJECT-STRUCTURE.md
```

**Önemli:** `assets/` build pipeline'a girmez; Express veya statik hosting ile doğrudan sunulur. Görsel/ses değiştirmek için `assets/` altındaki dosyaları replace etmek yeterli.

---

## 4. cooljs Motor Deseni

Proje, cooljs'in **Entity-Component benzeri Instance modelini** kullanır. Her oyun nesnesi:

```javascript
const obj = new Instance({
  name: 'unique_name',
  action: (instance, engine, time) => { /* logic per frame */ },
  painter: (instance, engine) => { /* draw per frame */ }
})
engine.addInstance(obj, layerName?) // opsiyonel layer
```

### Engine yaşam döngüsü (her frame)

```
requestAnimationFrame
  → tick (fps hesabı)
  → startAnimate(engine, time)      // src/animateFuncs.js — yeni blok, uçak spawn
  → paintUnderInstance(engine)      // src/background.js
  → updateInstances (her Instance.action)
  → paintInstances (her Instance.painter)
  → endAnimate(engine, time)        // HUD: skor, kat, kalpler
```

### Engine API (projede kullanılanlar)

| API | Kullanım |
|-----|----------|
| `addImg(name, path)` / `addAudio(name, path)` | Asset kaydı |
| `load(onReady, onProgress)` | Asset yükleme + progress callback |
| `init()` | Game loop başlat |
| `setVariable(key, val)` / `getVariable(key, default?)` | Global oyun state (string key) |
| `setTimeMovement(id, durationMs)` | Zaman tabanlı animasyon başlat |
| `getTimeMovement(id, [[from,to]], callback, opts)` | Easing ile interpolasyon |
| `checkTimeMovement(id)` | Hareket hâlâ aktif mi? |
| `pixelsPerFrame(value)` | FPS-bağımsız hız normalizasyonu |
| `touchStartListener` | Dokunma/tıklama handler atama |
| `addLayer` / `swapLayer` | Çizim sırası (flight layer arkada) |
| `highResolution: true` | Retina: canvas 2x, CSS boyut korunur |

### Time Movement sistemi

Animasyonlar manuel frame hesabı yerine **süre bazlı easing** ile yapılır:

```javascript
engine.setTimeMovement(constant.hookDownMovement, 500)  // 500ms'lik hareket
engine.getTimeMovement(
  constant.hookDownMovement,
  [[startY, endY]],
  (value) => { instance.y = value },
  { name: 'block', easing: 'linear' }
)
```

Movement ID'leri `src/constant.js` içinde tanımlıdır (`HOOK_DOWN_MOVEMENT`, `MOVE_DOWN_MOVEMENT`, vb.).

---

## 5. Oyun Akışı (End-to-End)

```
index.html yüklenir
  → gameWidth/gameHeight hesaplanır (min 1.5 aspect ratio)
  → TowerGame(option) çağrılır
  → game.load() → asset progress → updateLoading()
  → %100 + window.load → loading gizlenir, landing gösterilir
  → #start tıklanır → game.start()
       → tutorial instance'ları eklenir
       → GAME_START_NOW = true
  → startAnimate her frame yeni blok spawn eder (koşullar sağlanınca)
  → Kullanıcı dokunur → touchEventHandler
       → HOOK_UP movement, blok BEFORE_DROP
  → Blok düşer, line ile collision
       → success / perfect / rotate / miss
  → 3 miss → setGameFailed(3) → index.html modal gösterir
```

### Viewport hesabı (`index.html`)

```javascript
var ratio = 1.5
if (gameHeight / gameWidth < ratio) {
  gameWidth = Math.ceil(gameHeight / ratio)
}
```

Oyun alanı dikey ekranlarda ortalanır; CSS `rem` viewport yüksekliğine göre ölçeklenir (`html { font-size: 17.6vh }`).

---

## 6. Blok Durum Makinesi (FSM)

Blok durumları `src/constant.js` string sabitleriyle temsil edilir:

| Durum | Sabit | Açıklama |
|-------|-------|----------|
| Sallanma | `SWING` | Vinçle birlikte pendulum hareketi |
| Bırakma öncesi | `BEFORE_DROP` | Dokunma sonrası, düşüş başlamadan |
| Düşüş | `DROP` | Yerçekimi ile aşağı |
| İniş | `LAND` | Başarılı yerleşim; hafif yatay salınım |
| Dönerek düşüş | `ROTATE_LEFT` / `ROTATE_RIGHT` | Kısmen oturma, devrilme |
| Dışarı | `OUT` | Ekrandan çıktı, can kaybı |

### Çarpışma sonuçları (`block.js` → `checkCollision`)

```
0 = devam (henüz çarpışma yok)
1 = tamamen kaçırdı → düşmeye devam / OUT
2 = sol kenardan taştı → ROTATE_LEFT
3 = sağ kenardan taştı → ROTATE_RIGHT
4 = success (oturdu)
5 = perfect (±10% hizalama)
```

Perfect eşiği: blok merkezi, referans `line.x` ile `line.x + calWidth*0.8` … `line.x + calWidth*1.2` aralığında.

---

## 7. Modül Sorumlulukları

### `src/index.js` — TowerGame factory

- `Engine` oluşturur, tüm asset'leri `./assets/` path'i ile kaydeder
- Boyut sabitlerini ekrana göre hesaplar (`blockWidth = width * 0.25`, vb.)
- Kalıcı instance'lar: 4 bulut, line, hook
- `game.start()` ile tutorial ekler ve `GAME_START_NOW` açar
- `window.TowerGame = ...` global export

### `src/animateFuncs.js`

**`startAnimate`:** Oyun aktifken her frame:
- Son blok `land` veya `out` ise ve hareket yoksa → yeni `block_N` instance
- Rastgele `initialAngle` (zorluğa göre)
- `hookDownMovement` tetikler
- Belirli `successCount` değerlerinde uçak (`flight.js`) spawn

**`endAnimate`:** Canvas HUD — kat sayısı, skor, kalp ikonları (başarısızlık sayısına göre soluk)

### `src/utils.js` — Merkezi oyun kuralları

| Fonksiyon | Rol |
|-----------|-----|
| `touchEventHandler` | Dokunma → kanca yukarı, blok bırak |
| `getSwingBlockVelocity` | Pendulum hızı (`Math.sin`); kat arttıkça zorlaşır |
| `getLandBlockVelocity` | Oturan blokların yatay salınımı |
| `getAngleBase` | Yeni blok için max açı (30° → 60° → 80°) |
| `getMoveDownValue` | Kule yükseldikçe kamera aşağı kayma miktarı |
| `addScore` / `addSuccessCount` / `addFailedCount` | Skor & istatistik + option callback'leri |
| `drawYellowString` | Gradient + stroke'lu metin (wenxue font) |

### `src/line.js`

- İlk blok zemini: `lineInitialOffset` (background'dan türetilir)
- Her başarılı inişte `line.x`, `line.y`, `line.collisionX` güncellenir
- Debug modda kırmızı collision çizgisi

### `src/hook.js`

- Vinç görseli; blok ile aynı pendulum açısını paylaşır
- `hookUpMovement` / `hookDownMovement` ile Y pozisyonu

### `src/background.js`

- Dinamik linear gradient (kat arttıkça renk kayması)
- `background.png` parallax scroll
- 10. ve 15. katta `lightningMovement` flaş efekti

### `src/cloud.js` / `src/flight.js`

- Dekoratif parallax; oyun mekaniğine etki etmez
- Flight belirli kat eşiklerinde tek seferlik spawn

---

## 8. Global State (Engine Variables)

Tüm state `engine.setVariable(key, value)` ile tutulur. Key'ler `constant.js`'de:

| Key | Anlam |
|-----|-------|
| `GAME_START_NOW` | Oyun aktif mi |
| `GAME_USER_OPTION` | index.html'den gelen option objesi |
| `BLOCK_COUNT` | Spawn edilen blok sayısı |
| `SUCCESS_COUNT` | Başarılı kat (floor) |
| `FAILED_COUNT` | Kaçırılan blok (can) |
| `PERFECT_COUNT` | Ardışık perfect sayacı |
| `GAME_SCORE` | Toplam puan |
| `HARD_MODE` | Hile/zor mod (blok ekran dışına taşınca) |
| `INITIAL_ANGLE` | Mevcut blok pendulum açısı |
| `ROPE_HEIGHT` | İp uzunluğu (hard mode'da random) |
| `BLOCK_WIDTH` / `BLOCK_HEIGHT` | Blok boyutları |
| `LINE_INITIAL_OFFSET` | İlk zemin Y |

---

## 9. Özelleştirme Yöntemi

### A) Görsel / ses (`assets/`)

Dosya adını koruyarak replace edin. Kod path'leri `./assets/{filename}` formatında sabittir.

### B) Oyun kuralları (`index.html` → `option` objesi)

```javascript
const option = {
  width: gameWidth,
  height: gameHeight,
  canvasId: 'canvas',
  soundOn: true,
  successScore: 25,        // opsiyonel, varsayılan 25
  perfectScore: 25,        // opsiyonel, ardışık perfect bonusu
  hookSpeed: (floor, score) => number,
  hookAngle: (floor, score) => number,
  landBlockSpeed: (floor, score) => number,
  setGameScore: (score) => {},
  setGameSuccess: (count) => {},
  setGameFailed: (count) => { /* count >= 3 → modal */ }
}
```

Bu callback'ler `utils.js` içinden çağrılır; HTML kabuk skoru DOM'da göstermek için kullanır.

### C) Oyun mantığı (`src/`)

| Değişiklik | Dosya |
|------------|-------|
| Çarpışma toleransı | `src/block.js` → `checkCollision` |
| Zorluk eğrisi | `src/utils.js` → switch/case eşikleri |
| Yeni dekor | Yeni Instance modülü + `index.js`'de kayıt |
| HUD | `src/animateFuncs.js` → `endAnimate` |
| Spawn koşulları | `src/animateFuncs.js` → `startAnimate` |

Değişiklik sonrası **`npm run build`** zorunlu; HTML `dist/main.js`'i yükler.

---

## 10. Zorluk Progresyonu (Varsayılan)

| successCount | Etki |
|--------------|------|
| < 1 | Pendulum hızı 0 (ilk blok sabit) |
| < 10 | Açı tabanı 30°, orta hız |
| < 20 | Açı 60° |
| ≥ 20 | Açı 80° |
| ≥ 5 | Oturan bloklar yatay salınır |
| 2,6,8,14,18,22,25 | Uçak spawn |
| 10, 15 | Yıldırım flaşı |
| Blok x ekranın dış 1/3'ünde | `HARD_MODE = true` (açı 90°, hız artışı) |

Skor formülü (`addScore`):

```
perfectCount = isPerfect ? previousPerfect + 1 : 0
score += successScore + (perfectScore * perfectCount)
```

---

## 11. HTML Kabuk Entegrasyonu

| Bileşen | Teknoloji | Rol |
|---------|-----------|-----|
| Loading | CSS + jQuery-like Zepto | Asset % progress |
| Landing | CSS animasyon (`swing`, `slideTop`) | Start butonu |
| Modal | `#modal`, `#over-modal` | Game over + skor |
| Reload | `?s=timestamp` cache bust | `.js-reload` |

Oyun canvas başlangıçta `hide`; asset + DOM ready sonrası gösterilir.

**Script sırası önemli:**

```html
<script src="./dist/main.js"></script>   <!-- TowerGame tanımı -->
<script src="./assets/zepto-1.1.6.min.js"></script>
<script>/* init */</script>
```

---

## 12. AI İçin Pratik Kurallar

1. **Kaynak vs bundle:** Mantık değişikliği `src/` içinde yapılır; `dist/main.js`'i elle düzenlemeyin.
2. **Build unutma:** Her `src/` değişikliğinden sonra `npm run build`.
3. **State key'leri:** Yeni state için `constant.js`'e string sabit ekleyin; magic string kullanmayın.
4. **Instance isimlendirme:** Bloklar `block_${blockCount}`; benzersiz olmalı.
5. **FPS bağımsızlık:** Piksel hızları için `engine.pixelsPerFrame()` kullanın.
6. **Movement çakışması:** Aynı anda `hookUpMovement` varken yeni blok spawn edilmez (`startAnimate` kontrol eder).
7. **Debug:** Engine `debug: true` ile FPS + collision çizgileri; Enter ile pause (`index.js`'de listener var, factory'de debug flag set edilmiyor — gerekirse option'dan geçirin).
8. **Ses:** `soundOn: false` ile tüm audio devre dışı.
9. **Mobil:** cooljs touch/mouse birleşik `touchStartListener`; ayrı click handler gerekmez.
10. **Özelleştirme önceliği:** Davranış değişikliği için önce `option` hook'larını kontrol edin; core'a dokunmadan çözülebilir.

---

## 13. Bağımlılık Grafiği (Modüller)

```
index.html
  └── dist/main.js (webpack bundle)
        └── src/index.js
              ├── cooljs (Engine, Instance)
              ├── background.js → utils.js, constant.js
              ├── line.js → utils.js, constant.js
              ├── cloud.js → utils.js, constant.js
              ├── hook.js → utils.js, constant.js
              ├── animateFuncs.js → block.js, flight.js, utils.js, constant.js
              └── utils.js → constant.js

block.js → utils.js, constant.js
flight.js → cooljs Instance, constant.js
tutorial.js → utils.js, constant.js
```

---

## 14. Bilinen Kısıtlar

- Webpack config dosyası yok; entry/output varsayılan.
- `assets/` repo'da olmayabilir (gitignore veya ayrı indirme); oyun asset'ler olmadan çalışmaz.
- Babel/Webpack sürümleri eski (2018); Node 18+ için `--openssl-legacy-provider` gerekli.
- Test altyapısı yok.
- TypeScript yok; JSDoc minimal.

---

## 15. Hızlı Referans — Yeni Özellik Ekleme

**Örnek: Rüzgar efekti eklemek**

1. `src/wind.js` — `windAction`, `windPainter`
2. `src/index.js` — `game.addInstance(new Instance({...}))`
3. Gerekirse `constant.js`'e state key
4. `npm run build`

**Örnek: Can sayısını 5 yapmak**

1. `src/utils.js` → `addFailedCount` içinde `failed >= 3` eşiğini değiştir
2. `src/animateFuncs.js` → `endAnimate` kalp döngüsünü `<= 5` yap
3. `index.html` → `setGameFailed` eşiğini güncelle
4. `npm run build`

---

*Son güncelleme: proje kaynak kodu analizi üzerinden oluşturuldu.*
