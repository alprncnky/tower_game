# Asset kullanım rehberi

Her dosyanın ekranda nerede göründüğü ve hangi kod parçasından yüklendiği. Canvas oyunu `src/index.js` üzerinden `./assets/` altından asset kaydeder; HTML kabuğu (`index.html`) ayrı statik görseller kullanır.

---

## Canvas oyunu (oyun sırasında)

Oyun `#canvas` üzerinde çizilir. Katman sırası (alttan üste): arka plan gradyanı + `background.png` → bulutlar → uçan dekorlar → çizgi (debug) → kanca → bloklar → tutorial → HUD (`endAnimate`).

### Arka plan

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `background.png` | Tüm canvas genişliği; dikey kaydırma ile | Gökyüzü / şehir silüeti. Üstte renkli `linear-gradient` ile birleşir; kule yükseldikçe `moveDownMovement` ile aşağı kayar. Kaynak: `src/background.js` |

### Kanca ve ip

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `hook.png` | Üst orta; sallanırken döner | Kanca + ip tek sprite. Ekran ortasından (`width/2`) sarkar; `ropeHeight` kadar uzatılır, sallanma açısına göre rotate edilir. Kaynak: `src/hook.js` |

### Bloklar (kule parçaları)

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `block-rope.png` | Kancanın ucunda, sallanırken | Sallanma (`swing`) fazında: ip ucundaki asılı blok + ip görseli. `weightX` / `weightY` konumunda çizilir. Kaynak: `src/block.js` → `drawSwingBlock` |
| `block.png` | Kule üzerinde | Normal iniş / oturma / düşme / dönme durumlarında standart kule bloğu. Kaynak: `src/block.js` → `drawBlock` |
| `block-perfect.png` | Kule üzerinde | Merkeze çok yakın inişte (`perfect` çarpışma) aynı konumda `block.png` yerine kullanılır. |

### Dekor — bulutlar ve taşlar

Dört adet `cloud_*` instance’ı sürekli yatayda salınır; kule aşağı indikçe aşağı kayar. Yükseklik arttıkça taş sprite’larına geçilir (`count > 6`).

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `c1.png`, `c2.png`, `c3.png` | Üst yarıda, dört bölgeden rastgele | Erken oyunda bulut dekoru (sol/sağ, farklı yükseklikler). Kaynak: `src/cloud.js` |
| `c4.png` … `c8.png` | Aynı bulut katmanı | İlerleyen katta taş / kaya dekoru; aynı hareket mantığı. |

### Dekor — uçan objeler

Belirli kat sayılarında tek seferlik geçiş animasyonu (`src/animateFuncs.js` → `startAnimate`).

| Dosya | Tetiklenme (başarılı kat) | Hareket |
|-------|---------------------------|---------|
| `f1.png` | 2. kat | Soldan sağa |
| `f2.png` | 6. kat | Sağdan sola |
| `f3.png` | 8. kat | Soldan sağa |
| `f4.png` | 14. kat | Alttan yukarı |
| `f5.png` | 18. kat | Alttan yukarı |
| `f6.png` | 22. kat | Alttan yukarı |
| `f7.png` | 25. kat | Sağ üstten sola |

Kaynak: `src/flight.js`. Boyut `cloudSize` (~ekran genişliğinin %30’u).

### Tutorial (ilk dokunuş öncesi)

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `tutorial.png` | Sağ taraf, dikey ortanın biraz üstü (`~%45` yükseklik) | “Dokun” / nasıl oynanır ipucu metni veya ikon. Kaynak: `src/tutorial.js` |
| `tutorial-arrow.png` | Tutorial’ın hemen altında; hafif yukarı-aşağı animasyon | Parmağı / oku gösteren işaret. İlk dokunuşta her ikisi de kaldırılır (`src/utils.js`). |

### HUD (her karede, üst bant)

`src/animateFuncs.js` → `endAnimate` — oyun başladıktan sonra canvas üstünde:

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `score.png` | Sağ üst; genişlik ~%35 canvas | “Score” etiketi / çerçeve sprite’ı. Üzerine sayısal skor `drawYellowString` ile yazılır (`gameScore`, sağa hizalı). |
| `heart.png` | Skor etiketinin altında, 3 adet yan yana | Can / hata sayacı. 3 hak; her başarısız denemede bir kalp `globalAlpha: 0.2` ile soluklaşır (`failedCount`). |
| *(metin, asset değil)* | Sol üst | “floor” etiketi + `successCount` (kaç kat başarılı). Arial ile çizilir; 99+ katta konum kaydırılır. |

Skor ve kalp konumları (canvas koordinatı, `engine.width` tabanlı):

- Kat sayısı: sol ~%22–24, üst ~%12–20
- `score.png`: x ~%61, y ~%3.8
- Skor rakamı: sağ ~%90, üst ~%11
- Kalpler: x ~%66’dan başlayarak, y ~%16

---

## Ses (canvas oyunu)

`src/index.js` yalnızca `.mp3` yükler. Aynı isimli `.ogg` dosyaları repoda yedek / tarayıcı uyumluluğu içindir; mevcut kodda referans yok.

| Dosya | Ne zaman çalar |
|-------|----------------|
| `bgm.mp3` | Oyun yüklendikten sonra döngüde arka plan müziği (`playBgm`). Landing’de Start’a basınca tekrar başlatılır. |
| `drop.mp3` | Blok kuleye normal oturduğunda |
| `drop-perfect.mp3` | Mükemmel hizalı inişte |
| `rotate.mp3` | Blok kenardan düşüp dönmeye başladığında |
| `game-over.mp3` | 3. hata sonrası oyun bittiğinde (`addFailedCount`) |

---

## HTML kabuğu (canvas dışı DOM)

`index.html` + inline CSS. Oyun canvas’ı başta `hide`; yükleme bitince landing, 3 hatada modal açılır.

### Sayfa geneli

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `favicon.png` | Tarayıcı sekmesi | Site ikonu (`<link rel="icon">`) |
| `main-bg.png` | `body` arka planı, tekrarlı | Turuncu-kırmızı dış çerçeve / letterbox alanı (oyun alanı dışındaki boşluklar) |
| `wenxue.eot`, `.woff`, `.ttf`, `.svg` | Metin olarak | Özel font; yükleme yüzdesi, modal skor (`#score`), `.font-wenxue` sınıfı |

### Yükleme ekranı (`.loading`)

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `main-loading.gif` | Orta, genişlik ~%60 | Yükleme animasyonu |
| *(CSS bar)* | GIF altında | Beyaz çerçeveli ilerleme çubuğu; yüzde metni `wenxue` fontu |

### Ana menü / landing (`.landing`)

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `main-index-title.png` | Üst bölüm (`.action-1`), `swing` animasyonu | Oyun logosu / başlık |
| `main-index-start.png` | Alt orta (`.action-2`), genişlik ~%65 | “Start” düğmesi; tıklanınca landing kaybolur, `game.start()` çalışır |

### Oyun bitti modalı (`#modal`)

3 başarısız denemeden sonra (`setGameFailed` → `overShowOver`).

| Dosya | Ekrandaki konum | Açıklama |
|-------|-----------------|----------|
| `main-modal-bg.png` | Modal kartının arka planı | Dekoratif panel |
| `main-modal-over.png` | Kart üstü, “Game Over” görseli | Başlık / illüstrasyon |
| `main-modal-again-b.png` | Skor altında | “Try Again” / yeniden oyna düğmesi (`.js-reload` → sayfa yenileme) |
| *(DOM `#score`)* | `over-score` alanı | Son skor; `wenxue` ile yazılır (görsel asset değil) |

---

## Script

| Dosya | Kullanım |
|-------|----------|
| `zepto-1.1.6.min.js` | DOM seçiciler, yükleme/landing/modal geçişleri, Start ve reload tıklamaları (`index.html` inline script) |

---

## Repoda var, aktif kodda kullanılmıyor

Muhtemelen eski WeChat / paylaşım sürümünden kalan veya CSS’te hazır bırakılmış öğeler. Silmeden önce tasarım ihtiyacını kontrol edin.

| Dosya | Not |
|-------|-----|
| `rope.png` | İp için ayrı sprite; oyunda ip `hook.png` ve `block-rope.png` ile çiziliyor |
| `main-index-logo.png` | CSS’te `.landing .logo` tanımlı; `index.html` içinde `<img>` yok |
| `main-loading-logo.png` | CSS’te `.loading .logo` tanımlı; HTML’de logo alanı yok |
| `main-modal-invite-b.png` | Davet / paylaş düğmesi; mevcut modalda yok |
| `main-share-icon.png` | Paylaşım ikonu; referans yok |
| `bgm.ogg`, `drop.ogg`, `drop-perfect.ogg`, `game-over.ogg`, `rotate.ogg` | `addAudio` yalnızca `.mp3` kullanıyor |

---

## Hızlı referans: dosya → kod

| Asset | Yükleme / kullanım |
|-------|-------------------|
| Canvas görselleri | `src/index.js` → `game.addImg` |
| Canvas sesleri | `src/index.js` → `game.addAudio` |
| HUD | `src/animateFuncs.js` → `endAnimate` |
| HTML görselleri | `index.html` `<img>` / CSS `background` / `url()` |
| Font | `index.html` `@font-face` |

Dosya adını değiştirirken hem `src/index.js` hem `index.html` hem de bu listedeki referansları güncellemeniz gerekir; build sonrası `dist/main.js` yenilenmelidir (`npm run build`).
