# Curated adaylar (soft modern)

Bu klasördeki dosyalar **henüz oyuna bağlı değil**. `assets/` içine koymadan önce boyut/oran ayarla.

## gameplay/

| Dosya | Kenney kaynağı | Olası kullanım |
|-------|----------------|----------------|
| `background_blue_grass.png` | Platformer / Backgrounds | `background.png` temeli |
| `block_brickGrey.png` | Platformer / Tiles | `block.png` |
| `block-perfect_gemBlue.png` | Platformer / Items | `block-perfect.png` |
| `heart_hudHeart_full.png` | Platformer / HUD | `heart.png` |
| `tutorial-arrow_blue_s.png` | UI Pack Blue | `tutorial-arrow.png` |
| `flight_*.png` | Platformer / Enemies | `f1.png` … `f7.png` |

## ui/

Modal ve landing için `button_*`, `panel_*` — HTML’deki `main-modal-*.png` yerine Figma’da birleştir.

## audio/

`*.ogg` — UI tıklama. Oyun `mp3` bekliyor; örnek dönüşüm:

```bash
# ffmpeg gerekir
for f in curated/audio/*.ogg; do
  ffmpeg -i "$f" -y "curated/audio/$(basename "$f" .ogg).mp3"
done
```

`drop` / `bgm` / `game-over` için ayrıca kısa SFX araması gerekebilir (UI Pack’te sadece 6 UI sesi var).
