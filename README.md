# Regula — Portal Pencarian Peraturan Indonesia

> Portal full-stack pencarian, baca, dan kelola peraturan perundang-undangan Indonesia dengan PDF viewer dan highlight per-kata.

---

## 🚀 Cara Menjalankan

### Prasyarat
- Node.js ≥ 18
- npm ≥ 9
- PowerShell (Windows)

### Langkah Pertama

```powershell
cd C:\Users\DJTR\.gemini\antigravity\scratch\portal-peraturan

# Install dependensi
npm.cmd install

# Seed data awal (14 peraturan + 18 tema)
npm.cmd run seed

# Jalankan server
npm.cmd start
```

Buka browser di **http://localhost:5173**

### Build Produksi (opsional)
```powershell
npm.cmd run build
npm.cmd start
```

---

## 🔑 Akun Demo

| Role | Username | Password | Akses |
|------|----------|----------|-------|
| **Admin** | `admin` | `admin123` | Semua fitur + panel admin |
| **Editor** | `editor1` | `editor123` | Upload, moderasi, edit dokumen |
| **Pengguna** | `user1` | `user123` | Baca, bookmark, search history |
| **Pengunjung** | *(tanpa login)* | — | Hanya dokumen publik |

Atau gunakan tombol **"Demo Cepat"** di modal login untuk masuk instan.

---

## ✨ Fitur Utama

### 🔍 Mesin Pencarian Full-Text
- **Stemming Bahasa Indonesia** — query "perlindungan" cocok dengan "lindungi", "melindungi", dst
- **Operator Boolean** — `AND`, `OR`, `NOT` (contoh: `data pribadi AND kesehatan`)
- **Kutipan Tepat** — `"data pribadi"` (tanda kutip ganda)
- **Toleransi Typo** — "perlindunan" tetap menemukan "perlindungan"
- **Autocomplete** — saran kata saat mengetik di search bar
- **Faceted Filter** — filter berdasarkan Jenis, Visibilitas, Tema
- **Grup Hasil** — tampilkan flat atau dikelompokkan per tema

### 📄 PDF Viewer Canggih
- **Render Canvas** — PDF ditampilkan via pdf.js langsung di browser
- **Highlight Per-Kata** — kotak overlay kuning/oranye tepat di atas kata yang cocok query
- **Navigasi Keyboard**:
  - `/` — fokus ke search bar
  - `N` — kemunculan berikutnya
  - `P` — kemunculan sebelumnya
  - `G` — lompat ke halaman tertentu
- **Sidebar Kemunculan** — daftar semua kemunculan kata di dokumen, klik untuk lompat
- **OCR Fallback** — banner peringatan jika PDF adalah scan tanpa text layer

### 🗂️ Manajemen Tema Hierarkis
- 18 tema hukum berjenjang (Privasi & Data → Perlindungan Data Pribadi → dst)
- Tree sidebar dengan expand/collapse
- Jumlah dokumen per tema ditampilkan

### 📤 Upload & Pemrosesan
- **Drag & Drop** atau klik untuk upload PDF
- **Duplicate Detection** — cek hash SHA-256 sebelum simpan
- **Progress Stepper Real-time** — Validasi → Ekstraksi → Indexing → Selesai
- **Simulasi OCR** — toggle untuk test dokumen scan
- **"Buat Contoh PDF"** — generate PDF contoh langsung di browser (via pdf-lib)

### 👤 Role-Based Access Control
- **Pengunjung** — hanya dokumen publik
- **Pengguna** — bookmark, history, rating
- **Editor** — upload, edit metadata, replace versi
- **Admin** — semua + panel admin

### 🛡️ Panel Admin
- **Moderasi** — antrean dokumen diajukan, approve/reject
- **Manajemen Tema** — tambah/edit tema hierarkis
- **Antrean Proses** — monitor upload yang sedang diproses
- **Manajemen Pengguna** — lihat, ubah role, nonaktifkan akun
- **Analitik & Audit Log** — KPI (total dok, passage, avg search ms), log semua aktivitas

### 📌 Fitur Tambahan
- **Bookmark** — tandai dokumen favorit (tersimpan per user)
- **Riwayat Pencarian** — semua query tersimpan otomatis
- **Salin Sitasi** — format APA/Chicago/Bluebook dalam satu klik
- **Versi Dokumen** — riwayat revisi, bisa replace versi aktif
- **Umpan Balik** — rating bintang per dokumen
- **Keyboard Shortcut** global — tekan `?` untuk melihat daftar

---

## 🏗️ Arsitektur

```
portal-peraturan/
├── server/
│   ├── index.ts          # Express 5 API server
│   ├── searchEngine.ts   # Indonesian stemmer + scoring
│   ├── pdfProcessor.ts   # pdf-lib generator + pdfjs extractor
│   ├── store.ts          # In-memory store + JSON persistence
│   ├── seedData.ts       # 14 peraturan seed + 18 tema
│   └── seed.ts           # Seed runner script
├── src/
│   ├── api/client.ts     # Typed API client
│   ├── components/       # ShellLayout, AuthModal, HelpModal, ThemeIcon
│   └── pages/            # Home, Search, Detail, PDF, Upload, Admin, ...
├── shared/
│   └── types.ts          # TypeScript types shared FE+BE
├── server/storage/pdfs/  # PDF files (auto-generated on seed)
├── server/data/store.json # Persistent state
└── dist/                 # Built frontend (npm run build)
```

**Tech Stack:**
- **Frontend:** React 18, TypeScript, Vite, Tailwind CSS v4
- **Backend:** Express 5, tsx (TypeScript runner)
- **PDF:** pdfjs-dist (render + extract), pdf-lib (generate)
- **Search:** Custom engine — Indonesian stemmer, TF-IDF scoring, BM25-like ranking
- **State:** In-memory Map + JSON file persistence (no database)

---

## 🔧 API Endpoints Utama

| Method | Path | Deskripsi |
|--------|------|-----------|
| POST | `/api/auth/login` | Login |
| POST | `/api/auth/register` | Registrasi |
| GET | `/api/auth/me` | Info user aktif |
| GET | `/api/themes` | Daftar tema hierarkis |
| GET | `/api/search?q=...` | Full-text search + facets |
| GET | `/api/documents` | Katalog dokumen |
| GET | `/api/documents/:id` | Detail dokumen |
| GET | `/api/documents/:id/file` | Stream file PDF |
| GET | `/api/documents/:id/pages/:n/words` | Koordinat kata per halaman |
| GET | `/api/documents/:id/search?q=...` | Cari dalam dokumen |
| POST | `/api/upload` | Upload PDF baru |
| GET/POST | `/api/bookmarks` | Kelola bookmark |
| GET | `/api/admin/*` | Endpoint admin |

---

## 🐛 Troubleshooting

**Server tidak mau start:**
```powershell
# Cek apakah port 5173 sudah dipakai
netstat -ano | findstr :5173
# Kill proses jika perlu
Stop-Process -Id <PID>
```

**PDF tidak muncul / 404:**
```powershell
# Pastikan PDF sudah ada
Get-ChildItem "server\storage\pdfs\"
# Jika kosong, re-seed:
Remove-Item "server\data\store.json" -ErrorAction SilentlyContinue
npm.cmd run seed
npm.cmd start
```

**Data hilang setelah restart:**
- Cek `server/data/store.json` — ini file persistensi
- Jika rusak/kosong, hapus dan jalankan `npm.cmd run seed` lagi

**Highlight PDF tidak muncul:**
- Pastikan query diisi di URL param `?q=...`
- Beberapa PDF seed mungkin terlalu kecil untuk koordinat kata yang akurat
- OCR PDF (Perda Transjakarta) hanya punya fallback highlight

---

*Dibuat dengan Regula Portal v1.0 — November 2026*
