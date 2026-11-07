# Video Editor - Programmatic Video Studio

Studio programmatic video editing vertikal (9:16 Shorts/TikTok/Reels) berbasis **Vite + React 19 + Tailwind CSS v4 + Remotion** yang terintegrasi langsung dengan aset momen dari Google Drive.

---

## Fitur Utama

- **Live Interactive Studio Player (`@remotion/player`)**: Pratinjau animasi kartu, efek flash, transisi 3D flip, font, dan suara secara real-time tanpa kompilasi FFmpeg manual.
- **Meme Discovery System**:
  - **Kartu Meme 2:3 (Animal Kaiser / Pokemon)**: Frame 1:1, efek holographic rainbow foil sheen, aksen sudut mewah, dan rotasi 3D flip.
  - **Animasi Mirror Horisontal**: Gambar kartu di-mirror horizontal (`scaleX(-1)`) pada fase rotasi awal (`relFrame < 28`) dan kembali tegak normal saat kartu menghadap penonton, memberikan ilusi optik 3D yang realistis.
  - **Sistem 6 Rarity Konstan**: Common, Uncommon, Rare, Epic, Legendary, dan Mythic dengan integrasi kode warna tema otomatis dan header `<rarity> card discovered!`.
  - **Dua Mode Transisi (`enableScreenshot`)**:
    - **Mode Screenshot (Yes)**: Stop tepat di detik puncak $\rightarrow$ shutter flash kamera 10f $\rightarrow$ jeda 15f $\rightarrow$ reveal kartu 105f. Total durasi: `stopFrame + 130`.
    - **Mode Langsung (No)**: Untuk klip dengan facecam di awal; video latar belakang tetap berputar mulus (volume dibisukan saat kartu muncul), tanpa jeda/flash kartu langsung masuk di timestamp input, dan otomatis menahan frame akhir jika video fisik selesai lebih dulu. Total durasi: `stopFrame + 105`.
- **Zero-Dependency URL Query Routing**: Sinkronisasi state otomatis (`?moment=<slug>&style=<mode>`) via Web API `window.history` sehingga browser refresh (F5) tetap mempertahankan momen dan style yang sedang aktif.
- **1-Click MP4 Export & CLI Render**:
  - Tombol **Ekspor MP4** di web sidebar mengeksekusi render Remotion di latar belakang via endpoint `/api/render` dan langsung menyajikan berkas hasil render melalui rute `/out/*`.
  - Perintah CLI `npm run render` otomatis merender momen yang terakhir dibuka di `src/data.json`.
- **Google Drive Asset Integration**:
  - Sinkronisasi metadata otomatis via `scripts/gdrive.js` ke `src/data.json`.
  - Unduhan aset video (`full.mp4`) dan kartu (`image.jpg`) on-demand ke `public/temp/` dengan sanitasi path bebas spasi (`-`).

---

## Panduan Perintah

### 1. Menjalankan Studio Workbench (Browser)
```bash
npm run dev
```
Buka URL `http://localhost:5173` di peramban web.

### 2. Ekspor Video MP4 (CLI)
```bash
npm run render
```
Hasil video akan otomatis diekspor ke `out.mp4` berdasarkan momen dan mode aktif di `src/data.json`.

### 3. Mengunduh Video Momen dari Google Drive
```bash
npm run download
```
Pilih momen yang ingin diunduh asetnya ke `public/temp/`.

### 4. Sinkronisasi Metadata Momen dari Drive
```bash
npm run sync
```
Memperbarui daftar folder momen baru dari Google Drive ke `src/data.json`.

---

## Panduan Pengembangan AI Agent

Setiap instruksi arsitektur, catatan teknis, dan aturan kerja penting tersimpan di [GEMINI.md](file:///d:/projects/video-editor/GEMINI.md).
