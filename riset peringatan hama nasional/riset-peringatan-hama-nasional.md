
# Riset Sumber Data — Peringatan Hama Nasional
Tanggal: 2025-09-08 15:10

---

## 1) BBPOPT — Balai Besar Peramalan Organisme Pengganggu Tumbuhan (Ditjen Tanaman Pangan)
**Tipe sumber:** Halaman web WordPress (berita/arsip) + halaman layanan publik + (potensial) daftar surat kewaspadaan OPT.  
**URL yang dianalisis:**
- Beranda/portal BBPOPT: https://bbpopt.tanamanpangan.pertanian.go.id/
- Berita contoh (mengutip Sistem Peringatan Dini OPT & SIFORTUNA): https://bbpopt.tanamanpangan.pertanian.go.id/berita/cegah-ledakan-wereng-batang-coklat%2C-kementan-lakukan-gerdal-di-indramayu
- Halaman layanan publik “SURAT KEWASPADAAN OPT”: https://bbpopt.tanamanpangan.pertanian.go.id/c/ppid/informasi-setiap-saat/surat-kewaspadaan-opt/

**Status & Catatan:**
- Struktur tampak berbasis WordPress/Elementor. Konten berita statis (server-rendered), relatif mudah di-scrape.
- Halaman “Surat Kewaspadaan OPT” ada sebagai kategori/laman PPID; daftar file bisa bersifat dinamis/tersembunyi jika memerlukan JS, namun umumnya berupa tautan PDF/berkas.
- Selain berita, ada kategori **Aplikasi → SIFORTUNA** di dalam situs BBPOPT untuk navigasi menuju portal data.

**Selector CSS potensial:**
- Judul: `h1.entry-title`, `article.single .entry-title`, fallback: `article h1`
- Tanggal: `time.entry-date[datetime]`, fallback: `meta[property="article:published_time"]`
- Konten utama: `.entry-content`
- Kategori/Tag: `.cat-links a`, `.tags-links a`
- Lampiran PDF: `.entry-content a[href$=".pdf"]`
- Pada halaman indeks/arsip: `h2.entry-title a` (tautan ke detail)

**Kelayakan scraping:**
- **Mudah** untuk berita/artikel (SSR, cukup HTTP GET + parse DOM).
- **Sedang** untuk “Surat Kewaspadaan OPT” (perlu inspeksi apakah daftar dokumen dipublikasikan sebagai link langsung atau melalui JS).

---

## 2) SIFORTUNA — Sistem Informasi Forecasting OPT Nasional (BBPOPT/Dirjen Tanaman Pangan)
**Tipe sumber:** Portal/webapp (SPA), berisi prakiraan serangan OPT nasional dan fitur peringatan dini.  
**URL yang dianalisis:**
- Landing page: https://sifortuna.tanamanpangan.pertanian.go.id/
- Halaman masuk: https://sifortuna.tanamanpangan.pertanian.go.id/masuk
- Pustaka OPT (akses publik): https://sifortuna.tanamanpangan.pertanian.go.id/pustaka-opt
- Rilis berita peluncuran: https://bbpopt.tanamanpangan.pertanian.go.id/berita/luncurkan-aplikasi-sifortuna-bbpopt-serius-dukung-upaya-swasembada-pangan-yang-digagas-presiden-prabowo-dan-mentan

**Status & Catatan:**
- Webapp tampak **SPA (Javascript-heavy)**; beberapa rute publik ada (misal: “pustaka-opt”), namun data utama prakiraan/peringatan kemungkinan diambil via API internal.
- Rilis resmi menyatakan akses publik/gratis; tetapi halaman `/masuk` juga tersedia → kemungkinan ada **mode publik + akun** untuk fitur lanjutan.

**Selector CSS/strategi potensial (jika perlu headless browser):**
- Judul halaman: `head > title` (untuk penanda rute), atau `h1,h2` komponen utama.
- Kartu indikator: `.card .card-title`, `.card .card-value`
- Tabel daftar peringatan: `table thead th`, `table tbody tr`
- Tombol/tautan unduh (jika ada): `a[href*="download"]`, `a[href$=".csv"], a[href$=".json"]`
- **Strategi scraping:** gunakan **headless browser** (Playwright/Puppeteer) untuk memuat rute publik; inspeksi **Network** untuk menemukan endpoint API (misal `/api/*`, `/v1/*`) agar selanjutnya bisa di-pull tanpa headless.

**Kelayakan scraping:**
- **Sedang–Sulit** (SPA + kemungkinan autentikasi untuk data penuh). Data ringkas publik berpotensi bisa diambil via endpoint non-autentik jika tersedia.

---

## Kesimpulan Singkat
1. **Kandidat utama sumber peringatan hama nasional**:
   - **BBPOPT** (WordPress): berita + laman PPID **“Surat Kewaspadaan OPT”** — berpotensi berisi daftar PDF peringatan nasional/daerah → **paling siap di-scrape** (SSR, selector jelas).
   - **SIFORTUNA** (SPA BBPOPT): portal prakiraan/early warning lintas komoditas → **perlu pendekatan headless/API discovery** untuk ekstraksi otomatis.

2. **Langkah awal implementasi scraping**:
   - **Phase 1 (SSR)**: scraping arsip/berita BBPOPT + kategori **Surat Kewaspadaan OPT** (ambil: judul, tanggal, URL, ringkasan, link PDF).
   - **Phase 2 (SPA)**: audit SIFORTUNA/EWS SIPANTARA dengan headless untuk menemukan endpoint API publik (jika ada) → pindah ke **pull API** tanpa headless.
   - **Phase 3 (PDF ingestion)**: siapkan parser PDF untuk dokumen pendukung (kewaspadaan/prakiraan) + ekstraksi metadata standar.

3. **Skema metadata minimal yang direkomendasikan**:
   - `title`, `published_at`, `source` (BBPOPT/SIFORTUNA/EWS), `commodity`, `pest`, `region`, `severity/level`, `document_url` (PDF jika ada), `page_url` (HTML), `extracted_at`.

---

## Lampiran: Daftar Selector CSS (Ringkas)
- **WordPress/Artikel Detail**
  - Judul: `h1.entry-title`
  - Tanggal: `time.entry-date[datetime]`
  - Konten: `.entry-content`
  - Lampiran: `.entry-content a[href$=".pdf"]`
- **WordPress/Arsip (daftar)**
  - Item: `article.post`
  - Judul item: `h2.entry-title a`
  - Excerpt: `.entry-summary` / `.post-excerpt`
  - Pagination: `.pagination a` / `.nav-links a`
- **SPA (SIFORTUNA/EWS)**
  - Kartu indikator: `.card .card-title`, `.card .card-value`
  - Tabel peringatan: `table thead th`, `table tbody tr`
  - Tombol unduh: `a[href*="download"], a[href$=".csv"], a[href$=".json"]`

---

## Catatan Implementasi
- Untuk **SPA**, gunakan Playwright/Puppeteer dengan opsi `wait_for_network_idle` sebelum ekstraksi DOM.
- Simpan **HTML snapshot** (MHTML/HTML) dan **PDF snapshot** di storage (versi-arsip) untuk reproducibility.
- Bangun **normalizer** untuk memetakan variasi struktur ke skema metadata minimal di atas.