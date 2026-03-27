# 04. Backend Architecture - The Speed Engine (Bun + Elysia)

## Core Technology & Monorepo Mindset

- **Runtime**: **Bun** — runtime JavaScript native (pengganti Node.js) dengan performa native-like.
- **Framework**: **ElysiaJS** — framework web yang dibangun khusus untuk Bun, type-safe end-to-end.
- **ORM**: **Drizzle ORM** + **PostgreSQL** — query builder type-safe tanpa overhead.
- **Cache**: **Redis** — caching layout homepage, config global, dan RSS Feed agar ultra-cepat.
- **Type Bridge**: **Eden Treaty** (Bun package `@elysiajs/eden`) — menjembatani type API dari Backend ke Frontend Astro secara E2E.
- **Validation**: **TypeBox** — validasi skema request/response yang ketat di setiap endpoint.

---

## Route Architecture (Struktur Endpoint)

Semua rute dikelompokkan per domain sesuai Blueprint Database (14 Domain):

```
/api
├── /auth
│   ├── POST /login           → Login Admin (JWT)
│   ├── POST /public/login    → Login Masyarakat (JWT terpisah)
│   ├── POST /public/register → Registrasi Masyarakat (OTP WA)
│   └── POST /logout
│
├── /global
│   ├── GET  /config          → Singleton global_configs (PT/PN mode, logo, URL)
│   └── PATCH /config         → Update config (Super Admin only)
│
├── /layout
│   ├── GET  /homepage        → Seluruh layout_blocks aktif (dari Redis cache)
│   ├── PATCH /blocks/:id     → Update order/aktif blok
│   ├── GET  /menus           → Pohon navigasi_menus (recursive)
│   └── PATCH /menus/:id
│
├── /posts
│   ├── GET  /                → List (filter: type, category, tag, status, lang)
│   ├── POST /                → Buat berita baru (Editor)
│   ├── GET  /:slug           → Detail berita publik + increment view_count
│   ├── PATCH /:id/review     → Submit ke review (Editor → IN_REVIEW)
│   ├── PATCH /:id/approve    → Publish (Super Admin/PPID → PUBLISHED)
│   ├── PATCH /:id/reject     → Tolak dengan rejection_reason
│   ├── GET  /:id/revisions   → Riwayat versi post_revisions
│   └── POST /:id/share       → Increment share_count
│
├── /categories               → CRUD categories (hierarki dengan parent_id)
├── /tags                     → CRUD tags + autocomplete
├── /media
│   ├── POST /upload          → Upload file (validasi MIME: webp/pdf/mp4)
│   └── GET  /                → Library media_assets
│
├── /reports
│   ├── GET  /                → List laporan publik (filter: type, year, period)
│   ├── POST /                → Upload laporan baru + lampiran
│   ├── GET  /:id/attachments → Lampiran ganda per laporan
│   └── DELETE /:id
│
├── /staff                    → CRUD staff_members (Hakim, Pegawai, Role Model)
├── /institution              → CRUD institution_profiles (key-value CMS)
├── /sub-courts               → CRUD sub_courts (khusus mode PT)
│
├── /gallery
│   ├── GET  /                → List galleries (filter: type)
│   └── POST /items           → Upload item foto/video ke album
│
├── /tickets
│   ├── POST /                → Buat tiket baru (Masyarakat Login)
│   ├── GET  /my              → Tiket milik akun publik sendiri
│   ├── POST /:id/reply       → Balas tiket (Admin & Masyarakat)
│   └── PATCH /:id/status     → Update status tiket (Admin Layanan)
│
├── /forms
│   ├── GET  /:id             → Form builder publik
│   ├── POST /:id/submit      → Submit form (anonim/login)
│   └── GET  /:id/submissions → Hasil form untuk Admin (+ export Excel)
│
├── /zi
│   ├── GET  /status          → Status WBK/WBBM terkini
│   ├── GET  /areas           → 6 area ZI + progress
│   ├── POST /documents       → Upload dokumen bukti dukung per area
│   └── GET  /documents/:area
│
├── /ampuh
│   ├── GET  /certifications  → Riwayat sertifikasi AMPUH
│   ├── GET  /areas           → 7 area penilaian + skor
│   ├── GET  /evidence-items  → Ceklis item per area
│   ├── POST /evidences       → Upload bukti per item
│   └── PATCH /evidences/:id/verify → Verifikasi bukti (Admin)
│
├── /rb-activities            → CRUD kegiatan RB + foto
├── /custom-modules           → CRUD modul kustom (HTML/Script/Rich Text)
└── /stats
    ├── GET  /                → Statistik pengunjung
    └── POST /increment       → Catat pageview (internal counter)
```

---

## Security Middleware Stack

1. **`authGuard`** — Validasi JWT Header untuk semua rute `/api/admin/*`
2. **`publicAuthGuard`** — Validasi JWT masyarakat untuk rute tiket/form login
3. **`rbacCheck(role)`** — Cek role SUPERADMIN/EDITOR/ADMIN_LAYANAN per endpoint
4. **`rateLimiter`** — Batas 100 req/menit per IP (ElysiaJS plugin)
5. **`mimeValidator`** — Upload file divalidasi MIME type (tolak `.exe`, `.php`, `.sh`)
6. **`uriValidator`** — Semua field iframe URL divalidasi format `uri` via TypeBox

---

## Mode Multi-Tenant (PT / PN)

- Setiap request ke `/api/global/config` akan mengembalikan `tenant_mode`
- Astro membaca mode ini di SSR init untuk menyesuaikan:
  - Menu Kepaniteraan yang ditampilkan (`court_class` menentukan Tipikor/PHI dll)
  - Widget Sidebar PT: menampilkan `sub_courts` list
  - Widget Sidebar PN: menampilkan satu link `parent_court_url`

---

## Caching Strategy (Redis)

| Data | TTL | Trigger Invalidasi |
|---|---|---|
| `global_configs` | 5 menit | PATCH /global/config |
| `layout_blocks` (homepage) | 2 menit | PATCH /layout/blocks |
| `navigation_menus` | 10 menit | PATCH /layout/menus |
| RSS Feed MA/Badilum | 30 menit | Cronjob tiap 30 menit |
| List berita terbaru | 1 menit | POST/PATCH /posts |

---

## Backup & Maintenance

- **Auto Backup**: Container `pg-backup` Cron harian 02:00 AM (Docker Compose)
- **Manual Trigger**: `POST /api/admin/backup/trigger` → jalankan `pg_dump` via `Bun.$`
- **Maintenance Mode**: Jika `global_configs.maintenance_mode = true`, seluruh SSR Astro dialihkan ke halaman maintenance tanpa menyentuh kode
