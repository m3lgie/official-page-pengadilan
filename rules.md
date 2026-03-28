# RULES.MD — Panduan Komando Pembangunan Sistem Website Pengadilan 2026

# The Speed Stack: Astro + Bun + Elysia + Drizzle + PostgreSQL + Redis

> Dokumen ini adalah **hukum tertinggi** dalam pembangunan sistem ini.
> Setiap baris kode yang ditulis WAJIB berpedoman pada rules ini.
> Setiap agent, developer, dan AI assistant HARUS membaca dokumen ini sebelum memulai pekerjaan apapun.

---

## 📚 REFERENSI DOKUMEN (Otak Agen — Wajib Diketahui)

Semua keputusan teknis didasarkan pada 18 dokumen berikut di folder `.agent/docs/`:

| No  | File                                  | Isi Kunci                                                                    |
| --- | ------------------------------------- | ---------------------------------------------------------------------------- |
| 01  | `01_Analisa_Juknis_2026.md`           | Ketentuan SK DJU 127/2026: menu wajib, aksesibilitas, konten PT vs PN        |
| 02  | `02_Standar_Portal_Pemerintah.md`     | HA, SEO, WAF, Workflow Maker-Checker-Approver, SSO                           |
| 03  | `03_Frontend_Astro.md`                | Page Map 50+ halaman, komponen React Island, Tailwind + Shadcn/UI            |
| 04  | `04_Backend_Bun_Elysia.md`            | Seluruh endpoint API, middleware stack, Redis cache strategy                 |
| 05  | `05_Struktur_Folder.md`               | Hierarki monorepo: `/apps`, `/packages`, `/infra`                            |
| 06  | `06_Spek_Admin_Panel.md`              | RBAC, 2FA, modul CMS, disability support, audit trail                        |
| 07  | `07_Admin_Panel_Deep_Dive.md`         | Form builder, theme engine, script injection                                 |
| 08  | `08_Manajemen_Layout_Dinamis.md`      | Block-based homepage, sidebar widgets, RSS, IKM widget                       |
| 09  | `09_Aturan_Pengembangan.md`           | Conventional commits, no placeholders, JSDoc, image optimization             |
| 10  | `10_Panduan_Setup.md`                 | Instalasi Docker 3-langkah untuk setiap satuan kerja                         |
| 11  | `11_Rencana_Implementasi.md`          | 4 fase milestone pembangunan                                                 |
| 12  | `12_Strategi_Backup_Keamanan.md`      | SOP backup otomatis + manual trigger + restore playbook                      |
| 13  | `13_Skema_Database_Drizzle.md`        | **Blueprint 14 Domain, 50+ Tabel** — referensi mutlak schema.ts              |
| 14  | `14_Standar_Layout_Template.md`       | Wireframe visual Juknis: header, nav, hero, 2-kolom, footer                  |
| 15  | `15_Responsive_Design.md`             | Breakpoint system, perilaku layout per perangkat, mobile components, testing |
| 16  | `16_Manajemen_Struktur_Organisasi.md` | Hierarki jabatan PT/PN, filter dinamis posisi jabatan                        |
| 17  | `17_Analisis_Kendala_Mitigasi.md`     | Bottlenecks: React Island Hydration, Type-Safety dinamis, Cache Invalidation |
| 18  | `18_Arsitektur_Keamanan_Tinggi.md`    | Perisai Nginx, Dual JWT, Proteksi Web Shell (MIME bytes), SSRF               |

---

## ⚙️ ATURAN TEKNOLOGI (Tidak Boleh Diganti)

```
FRONTEND  : Astro (SSR mode, Bun adapter) + React Islands + Tailwind CSS + Shadcn/UI
BACKEND   : Bun runtime + ElysiaJS + Eden Treaty (E2E type-safe)
DATABASE  : PostgreSQL 15 + Drizzle ORM (pgTable, pgEnum, relations)
CACHE     : Redis 7 + Argon2 (password hashing)
VALIDASI  : TypeBox (semua request/response wajib tervalidasi)
CONTAINER : Docker Compose (lihat docker-compose.yml di root)
PROXY     : Nginx (satu-satunya titik masuk publik)
UI KIT    : Shadcn/UI (wajib untuk semua komponen interaktif admin)
STYLING   : Tailwind CSS (wajib) + CSS Variables (untuk theme engine)
BAHASA    : TypeScript STRICT mode di semua file .ts/.tsx/.astro
```

**Dilarang keras menggunakan:**

- ❌ `npm`, `yarn`, `pnpm` — gunakan **`bun`** saja
- ❌ `any` type di TypeScript — gunakan type eksplisit atau TypeBox schema
- ❌ Vanilla/custom CSS jika solusi ada di Tailwind/Shadcn
- ❌ `console.log` di production — gunakan Bun logger
- ❌ Framework lain (Next.js, Remix, Hono, Express) — sudah ditetapkan ElysiaJS

---

## 🏛️ ATURAN MULTI-TENANT (PT / PN)

> **PRINSIP INTI**: Satu codebase, konfigurasi berbeda di setiap satuan kerja.

1. **DILARANG** hardcode nama pengadilan, logo, atau warna di dalam kode.
2. Semua identitas dibaca dari tabel `global_configs` via API `/api/global/config`.
3. Mode dipilih via `tenant_mode`: `PENGADILAN_TINGGI` atau `PENGADILAN_NEGERI`.
4. `court_class` (`IA_KHUSUS`, `IA`, `IB`, `II`) menentukan menu Kepaniteraan yang muncul:
   - Tipikor → hanya jika `IA_KHUSUS` atau `IA`
   - PHI, Niaga, Perikanan → hanya jika kelas sesuai
5. Mode PT: tampilkan tabel `sub_courts` (daftar PN bawahan).
6. Mode PN: tampilkan link `parent_court_url` (PT pembina).

---

## 🗄️ ATURAN DATABASE (Drizzle ORM)

> Referensi mutlak: `13_Skema_Database_Drizzle.md` — **14 Domain, 50+ Tabel**

### Domain yang Harus Ada (Jangan ada yang terlewat):

1. **Pengguna & Otorisasi** — `users`, `public_accounts`
2. **Keamanan** — `audit_logs`
3. **Konfigurasi Global** — `global_configs` (singleton)
4. **CMS Berita & Konten** — `categories`, `tags`, `posts`, `post_tags`, `post_revisions`, `media_assets`
5. **Laporan Publik** — `public_reports`, `report_attachments`
6. **Profil Institusi** — `institution_profiles`, `staff_members`
7. **Layanan Masyarakat** — `service_types`, `public_tickets`, `ticket_replies`
8. **Form Builder** — `dynamic_forms`, `form_fields`, `form_submissions`
9. **Tata Letak** — `hero_slides`, `layout_blocks`, `navigation_menus`
10. **Modul Kustom** — `custom_modules` (Rich Text / RAW_HTML / EXTERNAL_SCRIPT)
11. **Sub-Pengadilan** — `sub_courts` (khusus mode PT)
12. **Statistik** — `site_stats`
13. **Reformasi Birokrasi** — `zi_status`, `zi_areas`, `zi_documents`, `ampuh_certifications`, `ampuh_kriteria`, `ampuh_evidence_items`, `ampuh_evidences`, `rb_activities`, `rb_activity_photos`
14. **Galeri Media** — `galleries`, `gallery_items`

### Aturan Drizzle:

- Gunakan `pgTable()` dan `pgEnum()` dari `drizzle-orm/pg-core`
- Semua UUID menggunakan `uuid('id').defaultRandom().primaryKey()`
- Semua tabel wajib memiliki `created_at: timestamp().defaultNow()`
- Definisikan semua relasi menggunakan `relations()` dari Drizzle
- Migrasi menggunakan `bun run db:push` (development) atau `db:migrate` (production)

---

## 🌐 ATURAN FRONTEND (Astro + React + Shadcn)

> Referensi: `03_Frontend_Astro.md` dan `14_Standar_Layout_Template.md`

### Arsitektur Routing (MPA + View Transitions):

- **Bukan SPA Konvensional**: Kita menggunakan arsitektur hibrida MPA (Multi-Page App) yang dirender di server (SSR).
- **Wajib View Transitions**: Fitur `<ViewTransitions />` Astro wajib dihidupkan untuk memastikan navigasi pergeseran halaman selembut SPA murni (tanpa berkedip ulang/_white-flash_), namun mempertahankan struktur mutlak keunggulan SEO asli dari HTML SSR.

### Struktur Layout (Wajib sesuai Wireframe Juknis):

```
Header: Logo + Nama PT/PN (dinamis) + Badge WBK/WBBM
NavBar: 7 menu wajib (Beranda, Tentang, Layanan Publik, Layanan Hukum, Berita, Hubungi, RB)
Hero: Slider (maks 5 slide) + Grid 10 Tautan Cepat (2 baris x 5)
Body: Layout Kiri 70% (konten utama) + Kanan 30% (sidebar)
Footer: Copyright © tahun + Nama Pengadilan (dinamis)
```

### Halaman Wajib Ada (50+ halaman — lihat Page Map di 03_Frontend_Astro.md):

- Semua halaman publik, halaman `/akun/` (masyarakat), dan halaman `/admin/`
- Pastikan rute `/admin/ampuh` → 9 kriteria (BUKAN area)
- Pastikan rute `/admin/zi` → 6 area ZI

### Aturan Komponen:

- `.astro` files → Semua halaman statis publik, layout, header, footer
- `.tsx` files (React Island) → HANYA komponen interaktif:
  - `<DragDropLayoutEditor />` — layout builder
  - `<NavigationTreeEditor />` — menu tree
  - `<RichTextEditor />` — editor berita
  - `<ReportViewer />` — viewer laporan multi-mode
  - `<PublicLoginForm />` — login masyarakat + OTP
  - `<TicketTracker />` — tracking tiket publik
  - `<AMPUHEvidenceUploader />` — upload bukti 9 Kriteria AMPUH
  - `<ZIProgressDashboard />` — progress 6 Area ZI
  - `<FormBuilder />` — admin form builder
  - `<MediaLibrary />` — media asset manager
  - `<SiteStatsChart />` — statistik pengunjung

### Aturan SEO (Wajib setiap halaman):

- `<title>` unik dan deskriptif per halaman
- `<meta name="description">` maks 160 karakter
- `<meta property="og:image">` untuk halaman berita
- JSON-LD schema `GovernmentOrganization` di halaman utama
- Sitemap XML otomatis via `@astrojs/sitemap`

### Aturan Laporan (`display_mode`):

- `INLINE_VIEWER` → render `<iframe>` PDF atau `<PDFViewer>` Shadcn
- `DOWNLOAD_BUTTON` → `<a href="..." download>` dengan ukuran file
- `PREVIEW_CARD` → card thumbnail + judul + CTA
- `EMBED_LINK` → link eksternal `target="_blank"`

---

## 📱 ATURAN RESPONSIVE DESIGN

> Referensi lengkap: `15_Responsive_Design.md`

### Pendekatan: Mobile First

- Tulis style default untuk mobile, override untuk layar lebih besar
- Gunakan **Tailwind CSS Breakpoints**: `sm` (480px), `md` (768px), `lg` (1024px), `xl` (1280px)

### Perilaku Wajib per Elemen:

- **Navbar**: Mobile → Hamburger ☰ + Drawer | Desktop → Full horizontal
- **Hero Slider**: Mobile tinggi 200px | Desktop 450-500px
- **Quick Links**: Mobile `grid-cols-2` | Desktop `grid-cols-5` (sesuai wireframe Juknis)
- **Body Layout**: Mobile 1-kolom penuh | Desktop 2-kolom 70%/30%
- **Berita Grid**: Mobile 1-kolom | Tablet 2-kolom | Desktop 3-kolom
- **Sidebar**: Mobile pindah ke bawah konten | Desktop kolom kanan
- **Admin Sidebar**: Mobile icon-only + drawer | Desktop full 250px

### Touch & Interaction:

- Touch target minimum **44x44px** (WCAG 2.5.5)
- Slider mendukung **swipe gesture** di mobile
- Tidak ada horizontal scroll di mobile (`overflow-x: hidden`)
- Hover effect hanya di non-touch device (`@media (hover: hover)`)

### Lighthouse Mobile Target:

- Performance ≥ 90, Accessibility 100, SEO 100

### Komponen Mobile Khusus (React Island):

- `<MobileNav />` — hamburger + drawer navigasi
- `<HeroSlider />` — swipe gesture + dots pagination

---

## 🔧 ATURAN BACKEND (Bun + Elysia)

> Referensi: `04_Backend_Bun_Elysia.md`

### Middleware Stack (Urutan Wajib):

1. `rateLimiter` — 100 req/menit per IP
2. `authGuard` — validasi JWT admin
3. `publicAuthGuard` — validasi JWT masyarakat
4. `rbacCheck(role)` — cek SUPERADMIN / EDITOR / ADMIN_LAYANAN
5. `mimeValidator` — tolak `.php`, `.sh`, `.exe`, `.py` di upload
6. `uriValidator` — validasi format URL di semua field iframe

### Endpoint Wajib Ada:

- Semua 15+ grup endpoint sesuai `04_Backend_Bun_Elysia.md`
- `/api/global/config` → Wajib di-cache Redis 5 menit
- `/api/ampuh/kriteria` → Response menggunakan kata "kriteria" bukan "area"
- `/api/backup/trigger` → Hanya SUPERADMIN, jalankan `Bun.$` pg_dump

### Redis Cache Strategy:

| Endpoint               | TTL      | Invalidasi           |
| ---------------------- | -------- | -------------------- |
| `/api/global/config`   | 5 menit  | PATCH /global/config |
| `/api/layout/homepage` | 2 menit  | PATCH /layout/blocks |
| `/api/layout/menus`    | 10 menit | PATCH /layout/menus  |
| RSS Feed               | 30 menit | Cronjob              |
| List berita            | 1 menit  | POST/PATCH /posts    |

---

## 🎨 ATURAN ADMIN PANEL

> Referensi: `06_Spek_Admin_Panel.md` dan `07_Admin_Panel_Deep_Dive.md`

### RBAC Roles:

- **SUPERADMIN**: Kendali penuh, backup manual, user management, global config
- **EDITOR (PPID)**: Berita, laporan, galeri — dengan workflow review
- **ADMIN_LAYANAN**: Tiket masyarakat, jadwal, PTSP

### Fitur Wajib Admin:

1. **Theme Engine**: Ubah `primary_color` → langsung berubah via CSS Variables
2. **Layout Builder**: Drag & drop urutan blok homepage (`layout_blocks`)
3. **Menu Builder**: Tree editor multi-level `navigation_menus` dengan `visibility` PT/PN
4. **Form Builder**: Drag & drop `form_fields` untuk survey IKM/IPAK/Pengaduan
5. **AMPUH Dashboard**: Ceklis 9 Kriteria, upload `ampuh_evidences`, status verifikasi
6. **ZI Dashboard**: Progress 6 Area, upload `zi_documents` per area
7. **Backup Panel**: Tombol trigger manual + daftar file backup + download
8. **Audit Trail**: Tabel `audit_logs` — semua mutasi tercatat dengan `payload_before/after`

### Workflow Konten (Maker-Checker-Approver):

```
Staff membuat → status: DRAFT
    ↓
Editor review → status: IN_REVIEW
    ↓
PPID/Pimpinan approve → status: PUBLISHED
    ↓ (jika ditolak)
status: REJECTED + rejection_reason wajib diisi
```

---

## 🔒 ATURAN KEAMANAN

> Referensi: `02_Standar_Portal_Pemerintah.md` dan `12_Strategi_Backup_Keamanan.md`

1. **Password**: Semua hashed menggunakan **Argon2** — tidak ada MD5/SHA1/bcrypt
2. **JWT**: Admin dan Masyarakat menggunakan JWT secret **terpisah** (`JWT_SECRET` vs `PUBLIC_JWT_SECRET`)
3. **Upload**: Semua file disimpan dengan `stored_name` UUID — bukan nama asli user
4. **Upload Path**: File tersimpan di volume `pengadilan_uploads` — tidak executable
5. **HTTPS**: Wajib di semua environment production (SSL cert via Certbot)
6. **2FA**: Wajib aktif untuk semua akun Admin
7. **Audit Log**: Setiap create/update/delete di tabel penting WAJIB menulis ke `audit_logs`
8. **Rate Limit**: 30 req/menit untuk `/api/*`, 60 req/menit untuk halaman web
9. **CORS**: Hanya izinkan origin dari `SITE_URL` di `.env`
10. **Iframe**: URL SIPP/e-Court divalidasi format URI sebelum disimpan

---

## ♿ ATURAN AKSESIBILITAS (WCAG 2.1 — Wajib Lulus)

> Referensi: `01_Analisa_Juknis_2026.md`

1. **Alt Text**: WAJIB pada semua `<img>`, icon, dan grafik — termasuk `thumbnail_alt` di tabel `posts`
2. **Contrast Ratio**: Minimum 4.5:1 antara teks dan latar belakang
3. **Heading Hierarchy**: Satu `<h1>` per halaman, `<h2>` untuk seksi, `<h3>` untuk sub-seksi
4. **Keyboard Navigation**: Semua elemen interaktif dapat dijangkau hanya dengan Tab/Enter
5. **Screen Reader**: Gunakan `aria-label` untuk icon-only buttons
6. **Video**: Semua video layanan PTSP WAJIB badge `is_sign_language` — tandai video Juru Bahasa Isyarat
7. **Subtitle**: Semua video wajib closed caption / teks subtitle
8. **Font Scale**: Layout tidak rusak saat font diperbesar 200%
9. **Target Size**: Tombol minimum 44x44px (WCAG 2.5.5)

---

## 📦 ATURAN DOCKER & DEPLOYMENT

> Referensi: `10_Panduan_Setup.md` dan `12_Strategi_Backup_Keamanan.md`

### Stack Docker (6 Container):

```
nginx        → Reverse proxy + file statis uploads
web-portal   → Astro SSR (internal port 4000)
api-server   → Elysia API (internal port 9090)
db           → PostgreSQL 15
redis        → Redis 7 (dengan password auth)
pg-backup    → Auto backup harian 02:00 AM
```

### Aturan Volume (WAJIB Named Volume):

- `pengadilan_pgdata` → Data PostgreSQL
- `pengadilan_uploads` → File upload laporan/foto
- `pengadilan_backups` → File dump .sql.gz
- `pengadilan_redisdata` → Persistensi Redis

### Cara Deploy Satuan Kerja Baru (3 Langkah):

```bash
1. cp .env.example .env && nano .env   # Isi SITE_URL, DB_PASSWORD, JWT_SECRET
2. docker compose up -d --build        # Build + jalankan semua container
3. docker exec -it pengadilan_api sh -c "bun run db:push && bun run db:seed"
```

### Setelah Deploy — Konfigurasi via Admin Panel:

- Set mode PT/PN → masuk `/admin/pengaturan`
- Upload logo, isi nama pengadilan, kelas pengadilan
- Set warna primer (hex), URL SIPP, e-Court, dll
- **Tidak ada perubahan kode yang dibutuhkan!**

---

## 📝 ATURAN PENULISAN KODE (Coding Standards)

> Referensi: `09_Aturan_Pengembangan.md`

### Git Workflow:

- Branch: `main` (production), `develop` (staging), `feature/xyz` (dev)
- **Conventional Commits** WAJIB:
  - `feat:` → fitur baru
  - `fix:` → perbaikan bug
  - `docs:` → perubahan dokumentasi
  - `refactor:` → refaktor tanpa ubah fungsionalitas
  - `chore:` → konfigurasi/setup

### Kode:

- Semua fungsi Elysia route wajib memiliki **JSDoc** params & return type
- Semua React Island wajib memiliki **PropTypes via TypeBox schema**
- Dilarang gambar PNG/JPG mentah di production → konversi ke **WebP** via `astro:assets`
- Dilarang teks **"Lorem Ipsum"** atau placeholder data generik
- Data simulasi WAJIB menggunakan contoh nyata hukum Indonesia

### Environment:

- Semua secret disimpan di `.env` — **jangan commit `.env` ke Git**
- `.env.example` diupdate setiap ada variabel baru
- File `.gitignore` WAJIB mencakup: `.env`, `node_modules/`, `dist/`, `*.sql.gz`

---

## 🚨 LARANGAN MUTLAK (TIDAK BOLEH DILANGGAR)

1. ❌ **Hardcode** nama pengadilan, warna, atau URL di dalam source code
2. ❌ **Satu akun admin** untuk banyak orang (shared account) — harus individual
3. ❌ **Upload file executable** (`.php`, `.sh`, `.exe`, `.py`) oleh user
4. ❌ **Menampilkan identitas pihak berperkara** (nama, foto, alamat) yang bukan putusan terbuka
5. ❌ **Foto anak** tanpa penyamaran wajah — sesuai perlindungan data pribadi
6. ❌ **Konten politik, iklan komersial**, atau promosi pihak luar di portal pengadilan
7. ❌ **Google Analytics** — gunakan Matomo/Umami self-hosted
8. ❌ **JWT secret sama** untuk admin dan akun publik masyarakat
9. ❌ **Commit `.env`** ke repositori Git
10. ❌ **Deploy tanpa HTTPS** di environment production

---

## 🧠 ATURAN KNOWLEDGE SYNC (Pembaruan Otak Agen)

> **PRINSIP**: Ilmu baru yang ditemukan selama pengembangan WAJIB disinkronkan ke dokumen `.agent/docs/` yang relevan — tidak boleh hanya hidup di memori lokal percakapan.

### Kapan Wajib Update Dokumen:

| Situasi                               | Dokumen yang Diupdate                                       |
| ------------------------------------- | ----------------------------------------------------------- |
| Ditemukan terminologi resmi baru      | File MD yang menyebut terminologi tersebut                  |
| Skema database butuh tabel/kolom baru | `13_Skema_Database_Drizzle.md`                              |
| API endpoint baru ditambahkan         | `04_Backend_Bun_Elysia.md`                                  |
| Halaman/rute baru ditambahkan         | `03_Frontend_Astro.md`                                      |
| Fitur Admin Panel baru                | `06_Spek_Admin_Panel.md` atau `07_Admin_Panel_Deep_Dive.md` |
| Perubahan keamanan/backup             | `12_Strategi_Backup_Keamanan.md`                            |
| Perubahan prosedur instalasi          | `10_Panduan_Setup.md`                                       |
| Aturan coding baru disepakati         | `09_Aturan_Pengembangan.md`                                 |
| Layout/wireframe berubah              | `14_Standar_Layout_Template.md`                             |
| Interpretasi baru dari Juknis SK DJU  | `01_Analisa_Juknis_2026.md`                                 |
| Aturan baru                           | `rules.md` itu sendiri                                      |

### Cara Update yang Benar:

```
1. TEMUKAN ilmu/fakta baru (dari riset, koreksi user, atau penelusuran kode)
2. IDENTIFIKASI dokumen mana yang terdampak (bisa lebih dari 1 file)
3. UPDATE semua dokumen yang terdampak SEBELUM melanjutkan coding
4. COMMIT khusus: docs: update [nama-doc] — [ringkasan perubahan]
```

### Contoh Nyata yang Sudah Terjadi:

- ✅ User koreksi: AMPUH pakai "Kriteria" bukan "Area" → update `13_Skema_Database_Drizzle.md` + `03_Frontend_Astro.md` + `04_Backend_Bun_Elysia.md`
- ✅ Riset: Docker butuh container Nginx → update `docker-compose.yml` + `10_Panduan_Setup.md`
- ✅ Celah teridentifikasi: `media_assets` belum didefinisikan → update `13_Skema_Database_Drizzle.md`

> **JANGAN** simpan ilmu baru hanya dalam konteks percakapan. Jika percakapan berakhir atau berganti sesi, ilmu itu hilang. Dokumen `.agent/docs/` adalah **memori permanen** yang bertahan lintas sesi pengembangan.

---

## ✅ CHECKLIST SEBELUM MERGE KE `main`

- [ ] Lighthouse Score Desktop: Performance ≥ 95, Accessibility 100, SEO 100
- [ ] Lighthouse Score **Mobile**: Performance ≥ 90, Accessibility 100, SEO 100
- [ ] Tampilan benar di **375px** (mobile) dan **1280px** (desktop)
- [ ] Hamburger menu berfungsi di mobile
- [ ] Slider hero mendukung swipe gesture di mobile
- [ ] Semua gambar baru sudah WebP dan memiliki alt text
- [ ] Tidak ada `any` type di TypeScript
- [ ] Semua endpoint baru sudah tercover TypeBox validation
- [ ] `audit_logs` ditulis untuk semua mutasi data penting
- [ ] Tidak ada perubahan hardcode identitas satuan kerja
- [ ] Redis cache di-invalidate dengan benar setelah mutasi config/layout
- [ ] Docker compose test: `docker compose up -d --build` sukses dari nol
- [ ] Conventional commit message diterapkan
- [ ] **Semua ilmu/temuan baru sudah disinkronkan ke `.agent/docs/` yang relevan**

---

_Rules ini adalah dokumen hidup. Setiap penambahan fitur besar yang mengubah arsitektur wajib diikuti dengan update rules.md ini._
_Terakhir diperbarui: 2026-03-28 | Versi: 1.2 (Security & Mitigation Patch)_
