# Yeni tema: soft modern (Kenney)

## Hedef

Orijinal `assets/` dosyalarını Kenney CC0 paketleriyle değiştirerek yuvarlatılmış, pastel-mavi / gri tonlarda casual bir görünüm.

## İndirilen paketler (CC0)

| Paket | Kaynak | Dosya sayısı | Klasör |
|-------|--------|--------------|--------|
| UI Pack | [kenney.nl](https://kenney.nl/assets/ui-pack) · [OpenGameArt](https://opengameart.org/content/ui-pack) | ~1315 | `kenney/ui-pack/` |
| Platformer Pack Redux | [kenney.nl](https://kenney.nl/assets/platformer-pack-redux) · [OpenGameArt](https://opengameart.org/content/platformer-pack-redux-360-assets) | ~453 | `kenney/platformer-pack-redux/` |
| Game Icons | [kenney.nl](https://kenney.nl/assets/game-icons) · [OpenGameArt](https://opengameart.org/content/game-icons) | ~434 | `kenney/game-icons/` |
| Tiny Town | [OpenGameArt](https://opengameart.org/sites/default/files/kenney_tiny-town.zip) | ~141 | `kenney/tiny-town/` |
| UI Pack Adventure | [OpenGameArt](https://opengameart.org/content/ui-pack-adventure) | ~393 | `kenney/ui-pack-adventure/` |

Ham zip arşivleri: `downloads/`

## Seçilmiş adaylar

`curated/` altında oyuna yakın isimlerle kopyalar var. Tam eşleme için Figma/GIMP ile boyutlandırıp `assets/` dosya adlarına yeniden adlandır.

| Oyun dosyası | Önerilen aday (`curated/`) |
|--------------|----------------------------|
| `background.png` | `gameplay/background_blue_grass.png` veya `background_colored_grass.png` |
| `block.png` | `gameplay/block_brickGrey.png` veya `block_boxCrate.png` |
| `block-perfect.png` | `gameplay/block-perfect_gemBlue.png` |
| `heart.png` | `gameplay/heart_hudHeart_full.png` |
| `score.png` | `gameplay/score-icon_hudJewel_blue.png` |
| `tutorial-arrow.png` | `gameplay/tutorial-arrow_blue_s.png` |
| `f1`–`f7` | `flight_bee.png`, `flight_fishBlue.png`, `flight_fishPink.png`, … |
| `c1`–`c8` | `decor_*`, slime/coin varyantları (platformer `Enemies/`, `Items/`) |
| Modal / buton | `ui/button_*`, `ui/panel_*` |
| Sesler | `audio/*.ogg` → MP3’e çevir (`drop`, `bgm` için Mixkit de düşün) |

**Eksik (Kenney’de yok / elle yapılacak):** `hook.png`, `block-rope.png`, uzun dikey `background` kompoziti, `main-loading.gif`, landing başlık görselleri.

## Sonraki adımlar

1. `curated/reference/*_preview.png` dosyalarına bak, paleti onayla (Blue + Grey UI).
2. Blokları ~1:0.71 oranında kırp/ölçekle (`docs/ASSETS-PURPOSE.md`).
3. `src/background.js` `colorArr` gradient’ini mavi-yeşil pastel yap.
4. `index.html` arka plan rengini UI Grey/Blue ile uyumlu yap.
5. Onaylanan PNG’leri `assets/` içine aynı dosya adıyla kopyala → `npm start`.

Detaylı lisans ve indirme URL’leri: `SOURCES.md`.
