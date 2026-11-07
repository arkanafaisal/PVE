# Self-Correcting Rules & Project Knowledge Engine

Dokumen ini memuat seluruh rekap konteks arsitektur, riwayat keputusan teknis, fitur aktif, dan aturan kerja yang **WAJIB dipatuhi** oleh AI Agent saat berinteraksi di dalam repositori `video-editor`.

---

## 1. Identitas & Gambaran Proyek

- **Nama Proyek**: Video Editor (Programmatic Video Studio)
- **Lokasi Root**: `d:\projects\video-editor`
- **Tujuan**: Studio web dan engine kompilasi video terprogram (*programmatic video editing*) vertikal (9:16 Shorts/TikTok/Reels) berbasis Remotion, Vite, React 19, dan Tailwind CSS v4 yang terhubung langsung dengan Google Drive.
- **Status Arsitektur**: Proyek ini berdiri sendiri sebagai repositori mandiri (*decoupled*) setelah dipisahkan dari monorepo `clipping`.

---

## 2. Struktur Direktori & Peran Berkas

```
video-editor/
├── public/
│   ├── sfx/                     # Audio SFX (camera-shutter.webm, magical-reveal.webm)
│   └── temp/                    # Tempat video & gambar momen on-demand dari Drive (space-free: hyphen "-")
├── scripts/
│   └── gdrive.js                # Helper integrasi Google Drive (sync metadata, on-demand downloader)
├── src/
│   ├── remotion/
│   │   ├── Root.tsx             # Root komposisi Remotion (MemeDiscovery, RouletteQuiz, VerticalShorts)
│   │   └── index.ts             # Entry point pendaftaran komposisi untuk CLI Remotion
│   ├── styles/
│   │   ├── meme-discovery/
│   │   │   ├── MemeDiscoveryComposition.tsx  # Engine timeline sekuensial Meme Discovery
│   │   │   ├── components/
│   │   │   │   ├── PokemonCard.tsx           # Kartu meme 2:3 Animal Kaiser/Pokemon, flip 3D, mirror, foil
│   │   │   │   ├── ShutterFlash.tsx          # Efek flash screenshot kamera
│   │   │   │   └── RewindEffect.tsx          # (Legacy) Efek skip YouTube mundur
│   │   │   └── types.ts                      # Kamus RARITY_CONFIG, getRarityColor, MemeDiscoveryProps
│   │   └── roulette-quiz/
│   │       ├── RouletteQuizComposition.tsx   # Gaya video kuis roulette tebak momen
│   │       └── types.ts
│   ├── utils/
│   │   └── time.ts              # Parser format waktu (MM:SS.mmm ke detik)
│   ├── App.tsx                  # Web Studio Workbench (Dashboard, Live Player, Sidebar controls, Export)
│   ├── data.json                # Single source of truth metadata momen, konfigurasi kartu, dan active moment
│   ├── main.tsx                 # Entry point React
│   └── index.css                # Styling Tailwind v4 (@import "tailwindcss";)
├── package.json
├── remotion.config.ts           # Konfigurasi Remotion CLI (enable Tailwind, WebCodecs)
├── tsconfig.json
└── vite.config.ts               # Server Vite dengan embedded backend endpoints (/api/folders, /api/render, dll.)
```

---

## 3. Fitur Utama & Logika Alur Video

### A. Dua Mode Meme Discovery: Screenshot vs Langsung (`enableScreenshot`)
Pengguna dapat memilih mode eksekusi melalui switch **Yes / No** di sidebar:

1. **Mode Screenshot (`enableScreenshot = true` / Default)**:
   - Video diputar normal dari frame 0 hingga detik timestamp puncak (`stopFrame`).
   - Tepat di `stopFrame`: video membeku di frame puncak tersebut (`<Freeze active={frame >= stopFrame} frame={stopFrame - 1}>`).
   - Seketika terpicu efek flash putih kamera (10 frame) + SFX kamera (`camera-shutter.webm`).
   - Jeda pasca-screenshot 0.5 detik (15 frame).
   - Kartu meme meluncur dari bawah & 3D flip reveal (105 frame).
   - **Total Durasi Komposisi**: `stopFrame + 10 + 15 + 105 = stopFrame + 130 frame`.

2. **Mode Langsung (`enableScreenshot = false`)**:
   - Dirancang khusus untuk klip video yang frame lucunya (facecam) berada di awal, agar video tidak terpotong terlalu pendek.
   - Video di latar belakang **tetap terus berputar (*continuous background play*)** di balik overlay gelap (`bg-black/85`) dan kartu meme.
   - Kartu langsung mulai meluncur masuk tepat di frame timestamp input (`cardStartFrame = stopFrame`) **tanpa jeda, tanpa shutter flash, dan tanpa SFX kamera**.
   - Volume video otomatis dibisukan (0) begitu kartu mulai masuk agar suara video tidak bertabrakan dengan SFX *magical reveal* maupun pembacaan detail kartu.
   - **Penanganan EOF**: Jika durasi fisik video habis sebelum durasi kartu selesai, video otomatis menahan (*hold*) frame terakhirnya (`videoEndFrame - 1`) tanpa replay/loop.
   - **Penanganan Video Panjang**: Jika video lebih panjang dari kartu, video otomatis dipaksa berhenti tepat saat durasi show kartu selesai (`cardStartFrame + 105`).
   - **Total Durasi Komposisi**: `stopFrame + 105 frame`.

### B. Desain Kartu Meme 2:3 & Animasi 3D Flip
- **Rasio Kartu**: Minimalis Jimat 2:3 (lebar 760px, tinggi 1140px, rounded 38px) terinspirasi kartu koleksi *Animal Kaiser / Pokemon*.
- **Efek Mirror Horisontal Awal**:
  - Saat kartu mulai meluncur naik dan berputar pada sumbu Y (`relFrame < 28`, 0° -> 90°), elemen `<Img>` kartu diberi transformasi horizontal `scaleX(-1)`.
  - Saat kartu melewati sudut 90° dan berputar balik menghadap layar (`relFrame >= 28`, -90° -> 0°), orientasi gambar kembali normal (`transform: 'none'`) bersamaan dengan munculnya teks judul kartu, rarity, dan skill.
  - Ini menghasilkan ilusi optik 3D yang realistis seolah penonton melihat sisi balik/pantulan gambar kartu sebelum dibalik.
- **Holographic Rainbow Foil Sheen**: Efek kilau pelangi GPU-accelerated yang menyapu kartu begitu kartu selesai membalik (`relFrame >= 28`).

### C. Pembakuan 6 Tingkat Rarity
Tingkat kelangkaan kartu dibakukan ke dalam 6 konstanta dengan kode warna tema otomatis:
- **Common**: Abu-abu (`#9ca3af`)
- **Uncommon**: Hijau (`#22c55e`)
- **Rare**: Oranye (`#f97316`)
- **Epic**: Ungu (`#a855f7`)
- **Legendary**: Kuning / Emas (`#eab308`)
- **Mythic**: Merah (`#ef4444`)

Header teks di area hitam atas safe zone menampilkan format:
`<rarity> card discovered!` (contoh: *Mythic card discovered!*) dengan warna teks mengikuti warna heksadesimal rarity yang aktif.

### D. Zero-Dependency URL Query Routing
State momen dan mode tersinkronisasi otomatis ke URL menggunakan Web API bawaan (`window.history.pushState` / `replaceState` dan `popstate`):
`http://localhost:5173/?moment=<slug_or_id>&style=<mode>`
Ketika halaman di-refresh (F5), aplikasi tetap berada pada momen dan mode yang sedang diedit tanpa ter-reset ke dashboard.

### E. 1-Click Ekspor MP4 & CLI Render
- **Melalui Browser**: Tombol "Ekspor MP4" di sidebar web memicu endpoint internal `/api/render` pada `vite.config.ts`, yang mengeksekusi CLI Remotion di background menggunakan `{ shell: true }` (kompatibel dengan Windows). Hasil render disimpan di `out/` dan disajikan statis melalui rute `/out/*`.
- **Melalui Terminal**: Perintah `npm run render` otomatis membaca `activeMomentId` dari `src/data.json`, sehingga selalu merender momen yang terakhir dibuka.

---

## 4. Cara Menjalankan Studio & Skrip

1. **Menjalankan Web Studio**:
   ```bash
   npm run dev
   ```
   Buka `http://localhost:5173` di browser.

2. **Render Ekspor Video CLI**:
   ```bash
   npm run render
   ```

3. **Sinkronisasi Momen Google Drive ke data.json**:
   ```bash
   npm run sync
   ```

4. **Unduh Aset Momen Tertentu**:
   ```bash
   npm run download
   ```

---

## 5. Learned Rules (Aturan Wajib Agen AI)

Setiap AI Agent yang bekerja di repositori ini **WAJIB MEMATUHI** aturan berikut tanpa terkecuali:

1. **[PROCESS] Dilarang Menjalankan Full Video Build Saat Turn Berlangsung**:
   Jangan pernah mengeksekusi `npm run build` atau proses ekspor render video penuh (`remotion render` / `npm run render`) saat merespons percakapan pengguna, karena proses ini memakan waktu sangat lama dan membebani sistem. Selalu verifikasi perubahan langsung via pemutar interaktif live preview (`@remotion/player`).

2. **[PROCESS] Dilarang Menjalankan TypeScript Check (`tsc`)**:
   Jangan pernah mengeksekusi perintah cek tipe lambat seperti `npx tsc` atau `tsc --noEmit` dalam turn tanggapan kecuali diminta eksplisit oleh pengguna, karena `tsc` memakan latensi tinggi dan tidak mendeteksi isu rendering dinamis Remotion.

3. **[TOOL] Dilarang Menggunakan Headless Browser Automation**:
   Jangan gunakan browser automation (seperti Chrome DevTools MCP, Playwright, atau Puppeteer) untuk task coding sederhana, penyesuaian visual, atau pengeditan logika biasa karena menimbulkan latensi dan overhead koneksi yang berat.

4. **[ARCH] Dilarang Menggunakan Spasi pada Path Aset Lokal**:
   Jangan pernah menggunakan karakter spasi pada folder atau nama aset lokal di `public/temp/`. Selalu sanitasi path dengan mengganti spasi menjadi tanda hubung (`-`), misalnya `I-surrender` bukan `I surrender`, untuk mencegah error URL percent-encoding (`%20`), 404, dan crash pemutar video Remotion.

5. **[UX] Dilarang Menerapkan Auto Line-Break atau Dynamic Font-Scaling Acak**:
   Jangan menerapkan pemotongan baris teks otomatis yang merusak tinggi layout atau font-scaling dinamis tak terprediksi pada judul dan teks kartu; selalu pertahankan layout fixed-height atau hormati pemotongan baris manual dari user (`\n`).

6. **[DATA] Wajib Menggunakan Berkas `full.mp4`**:
   Saat membaca atau memproses aset video momen, selalu gunakan `full.mp4` sebagai berkas video penuh. Jangan pernah beralih ke berkas potongan parsial lain (seperti `cut.mp4`) untuk alur Meme Discovery.

7. **[TOOL] Aturan Version Control Git**:
   - Selalu lakukan `git add` dan `git commit` secara atomik per fitur dengan pesan commit yang jelas dan deskriptif.
   - **DILARANG KERAS** mengeksekusi `git push`, `git reset`, `git squash`, `git merge`, atau `git rebase` karena operasi remote dan modifikasi histori berada di bawah kendali penuh pengguna.
   - Jangan pernah menyertakan temporary files, debug logs, atau scratch files ke dalam commit Git.

8. **[PROCESS] Disclaimer Pertanyaan Pengguna**:
   Jika prompt pengguna diawali atau memuat disclaimer pertanyaan seperti *"ini saya bertanya ya"*, jawab secara analitis dan konseptual tanpa memodifikasi berkas kode apa pun.

9. [PROCESS] Always document each Git commit in changelog.md with its short commit ID, structural context, and key code diff explanations (what was removed/changed/added and why) immediately upon committing because commit messages alone lack diff context.

10. [PROCESS] Always stage and commit changelog.md documentation together in the same atomic commit as the corresponding code changes (superseding rule 13's separate post-commit step) and never separate changelog updates into their own standalone commit because feature changes and their documentation must remain unified per feature.

11. [PROCESS] Never perform post-commit re-checking or separate commits just to update short commit IDs in changelog.md; always record the commit as [Unreleased] or update previous commit IDs lazily in subsequent feature commits (superseding rules 13 and 15 regarding immediate commit ID resolution) because checking git status and amending or updating commit IDs in the same turn wastes valuable development time.