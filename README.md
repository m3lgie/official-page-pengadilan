<div align="center">

<img src="https://img.shields.io/badge/Status-In%20Development-orange?style=for-the-badge" />
<img src="https://img.shields.io/badge/Juknis-SK%20DJU%20127%2F2026-red?style=for-the-badge" />
<img src="https://img.shields.io/badge/License-Internal-gray?style=for-the-badge" />

# 🏛️ Official Page Pengadilan

### Portal Resmi Pengadilan Tinggi & Pengadilan Negeri

**Republik Indonesia — Mahkamah Agung**

_Dibangun sesuai Keputusan Direktur Jenderal Badan Peradilan Umum_
_Nomor: SK/DJU/127/HM1.1/III/2026_

---

[![Astro](https://img.shields.io/badge/Astro-5.x-FF5D01?style=flat-square&logo=astro&logoColor=white)](https://astro.build)
[![Bun](https://img.shields.io/badge/Bun-1.x-fbf0df?style=flat-square&logo=bun&logoColor=black)](https://bun.sh)
[![ElysiaJS](https://img.shields.io/badge/Elysia-1.x-7c3aed?style=flat-square)](https://elysiajs.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=flat-square&logo=redis&logoColor=white)](https://redis.io)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![TypeScript](https://img.shields.io/badge/TypeScript-Strict-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://typescriptlang.org)

</div>

---

## 📋 Tentang Proyek

**Official Page Pengadilan** adalah sistem portal web resmi berperforma tinggi yang dirancang untuk seluruh satuan kerja **Pengadilan Tinggi (PT)** dan **Pengadilan Negeri (PN)** di bawah Direktorat Jenderal Badan Peradilan Umum, Mahkamah Agung RI.

Sistem ini dibangun dengan filosofi **"satu codebase, konfigurasi berbeda"** — memungkinkan instalasi di ratusan satuan kerja di seluruh Indonesia hanya dengan mengubah file `.env` dan melakukan konfigurasi melalui Admin Panel, tanpa menyentuh kode sama sekali.

---

## ⚡ The Speed Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                     THE SPEED STACK                             │
│                                                                 │
│   🌐 FRONTEND          🔧 BACKEND           🗄️ DATA             │
│   ──────────────       ─────────────        ─────────────       │
│   Astro 5 (SSR)        Bun Runtime          PostgreSQL 15       │
│   React Islands        ElysiaJS             Drizzle ORM         │
│   Tailwind CSS         Eden Treaty          Redis 7             │
│   Shadcn/UI            TypeBox              pg-backup           │
│                                                                 │
│   🐳 INFRASTRUCTURE                                             │
│   ──────────────────────────────────────────────────────────    │
│   Nginx (Reverse Proxy) + Docker Compose (6 Containers)         │
└─────────────────────────────────────────────────────────────────┘
```

| Teknologi         | Peran                     | Keunggulan                                                            |
| ----------------- | ------------------------- | --------------------------------------------------------------------- |
| **Astro 5**       | Frontend SSR/SSG          | Zero-JS by default, Islands Architecture, View Transitions (SPA feel) |
| **Bun**           | Runtime & Package Manager | 3-4x lebih cepat dari Node.js                                         |
| **ElysiaJS**      | REST API Backend          | Type-safe E2E, native Bun, ultra-fast                                 |
| **Drizzle ORM**   | Database Query            | Type-safe SQL, migrasi otomatis                                       |
| **PostgreSQL 15** | Database Utama            | JSONB, Full-text Search                                               |
| **Redis 7**       | Cache Layer               | TTL cache, Auth password                                              |
| **Nginx**         | Reverse Proxy             | Rate limiting, SSL, static files                                      |

---

## 🎯 Fitur Utama

### 🏛️ Multi-Tenant (PT & PN)

- Satu instalasi mendukung mode **Pengadilan Tinggi** atau **Pengadilan Negeri**
- Identitas instansi (nama, logo, warna, URL layanan) dikelola via **Admin Panel**
- Struktur menu Kepaniteraan menyesuaikan **kelas pengadilan** (`IA Khusus`, `IA`, `IB`, `II`) otomatis

### 📰 CMS & Konten

- Editor berita **Rich Text** dengan workflow **Maker → Checker → Approver**
- Manajemen **Kategori bertingkat** & **Tag** dengan autocomplete
- **Revisi konten** tersimpan otomatis (riwayat lengkap)
- **Galeri Foto & Video** dengan album terorganisir
- **Laporan Publik** multi-format: inline PDF viewer, download, preview card, embed link

### 🎨 Layout Dinamis

- **Homepage Block-based**: drag & drop urutan widget tanpa coding
- **Hero Slider**: manajemen banner dengan CTA dan swipe gesture mobile
- **10 Tautan Cepat**: SIPP, e-Court, e-Berpadu, Direktori Putusan, dll
- **Sidebar Widgets**: RSS Feed MA/Badilum, IKM/IPAK, Jadwal Sidang (iFrame SIPP)
- **Navigation Builder**: menu bertingkat dengan visibilitas khusus PT/PN

### 🏗️ Struktur Organisasi Dinamis

- Bagan organisasi visual (tree chart) berbeda antara PT dan PN
- Jabatan kondisional berdasarkan kelas pengadilan (Tipikor, PHI, Niaga, Perikanan)
- Riwayat jabatan pegawai + SK pengangkatan
- Badge **Plt./Plh.** otomatis untuk pejabat sementara

### ⚖️ Reformasi Birokrasi

- **Zona Integritas (ZI)**: Dashboard progress 6 Area (WBK/WBBM) + upload dokumen bukti
- **AMPUH Badilum**: Ceklis 9 Kriteria penilaian resmi + upload evidence + verifikasi asesor
- **Kegiatan RB**: Dokumentasi foto kegiatan reformasi birokrasi

### 👥 Layanan Masyarakat

- **Login Masyarakat**: Registrasi mandiri dengan verifikasi OTP WhatsApp
- **Tiket Pengaduan**: Tracking status real-time dengan riwayat balasan
- **Form Builder**: Drag & drop pembuatan form survey (IKM, IPAK, Pengaduan)
- **Layanan Disabilitas**: Video Juru Bahasa Isyarat untuk setiap jenis layanan PTSP

### 🔒 Keamanan & Infrastruktur

- Password hashing dengan **Argon2** (bukan bcrypt/MD5)
- **JWT terpisah** untuk Admin dan akun Masyarakat
- **2FA wajib** untuk semua akun Admin
- **Audit Trail** lengkap (siapa mengubah apa, kapan)
- **Rate limiting** Nginx (30 req/mnt API, 60 req/mnt web)
- **Backup otomatis** harian jam 02:00 dengan retensi 7 hari / 4 minggu / 6 bulan

---

## 🗄️ Arsitektur Database

**14 Domain, 50+ Tabel** (Drizzle ORM + PostgreSQL 15)

```
Domain  1: Pengguna & Otorisasi    (users, public_accounts)
Domain  2: Keamanan                (audit_logs)
Domain  3: Konfigurasi Global      (global_configs — singleton)
Domain  4: CMS & Berita            (posts, categories, tags, post_revisions, media_assets)
Domain  5: Laporan Publik          (public_reports, report_attachments)
Domain  6: Profil & SDM            (institution_profiles, staff_members, org_positions, org_chart_nodes)
Domain  7: Layanan Masyarakat      (service_types, public_tickets, ticket_replies)
Domain  8: Form Builder            (dynamic_forms, form_fields, form_submissions)
Domain  9: Tata Letak              (hero_slides, layout_blocks, navigation_menus)
Domain 10: Modul Kustom            (custom_modules — HTML/Script/RichText)
Domain 11: Sub-Pengadilan          (sub_courts — khusus mode PT)
Domain 12: Statistik               (site_stats)
Domain 13: Reformasi Birokrasi     (zi_status, zi_areas, zi_documents,
                                    ampuh_certifications, ampuh_kriteria,
                                    ampuh_evidence_items, ampuh_evidences, rb_activities)
Domain 14: Galeri Media            (galleries, gallery_items)
```

---

## 🐳 Topologi Docker

```
Internet
    │
    ▼
[ Nginx :80/:443 ]  ← Satu-satunya titik masuk publik
    │
    ├──► [ web-portal :4000 ]  ← Astro SSR
    │         │
    │         └──► [ api-server :9090 ]  ← Bun + Elysia
    │                    │
    │               ┌────┴────┐
    │           [ db :5432 ] [ redis :6379 ]
    │
    └── [ pg-backup ]  ← Auto pg_dump 02:00 AM
```

---

## 🚀 Instalasi (3 Langkah)

### Prasyarat

- Docker v24+ & Docker Compose v2.20+
- Ubuntu 20.04+ (atau distro Linux lainnya)
- Port 80 dan 443 terbuka

### Setup

```bash
# 1. Clone repositori
git clone https://github.com/m3lgie/official-page-pengadilan.git
cd official-page-pengadilan

# 2. Konfigurasi environment
cp .env.example .env
nano .env  # Isi sesuai data pengadilan

# 3. Jalankan semua container
docker compose up -d --build

# 4. Inisialisasi database
docker exec -it pengadilan_api sh -c "bun run db:push && bun run db:seed"
```

### Konfigurasi `.env` Wajib Diisi

```env
SITE_URL=https://pn-namakota.go.id
DB_PASSWORD=password_kuat_minimal_20_karakter
REDIS_PASSWORD=password_redis_kuat
JWT_SECRET=   # generate: openssl rand -hex 32
PUBLIC_JWT_SECRET=   # generate: openssl rand -hex 32
```

### Setelah Deploy

Buka `https://domain-anda.go.id/admin/pengaturan` dan konfigurasikan:

- Mode PT/PN, kelas pengadilan, nama, logo, warna primer
- URL SIPP, e-Court, e-Berpadu, SIWAS

**Tidak perlu sentuh kode sama sekali! 🎉**

---

## 📁 Struktur Repository

```
official-page-pengadilan/
├── 📄 rules.md                    ← Panduan komando pembangunan (baca ini dulu!)
├── 📄 docker-compose.yml          ← Orkestrasi 6 container
├── 📄 .env.example                ← Template konfigurasi
├── 📁 apps/
│   ├── web-portal/                ← Astro SSR frontend
│   └── api-server/                ← Bun + ElysiaJS backend
├── 📁 packages/
│   ├── db/                        ← Drizzle schema & migrations
│   └── shared/                    ← Types & utilities bersama
├── 📁 infra/
│   ├── nginx/                     ← Konfigurasi Nginx
│   └── docker/                    ← Dockerfiles
└── 📁 .agent/docs/                ← 18 Dokumen Blueprint (Otak Agen)
    ├── 01_Analisa_Juknis_2026.md
    ├── 13_Skema_Database_Drizzle.md
    ├── 17_Analisis_Kendala_Mitigasi.md
    ├── 18_Arsitektur_Keamanan_Tinggi.md
    └── ... (18 dokumen total)
```

---

## 📖 Dokumentasi

Seluruh dokumentasi teknis tersimpan di folder `.agent/docs/`:

| #   | Dokumen             | Keterangan                       |
| --- | ------------------- | -------------------------------- |
| 01  | Analisa Juknis 2026 | Ketentuan SK DJU 127/2026        |
| 03  | Frontend Astro      | Page Map, React Islands, SEO     |
| 04  | Backend Elysia      | Endpoint API, Middleware, Redis  |
| 13  | Skema Database      | 14 Domain, 50+ Tabel Drizzle     |
| 14  | Standar Layout      | Wireframe referensi visual       |
| 15  | Responsive Design   | Breakpoint, Mobile-first         |
| 16  | Struktur Organisasi | Hierarki jabatan PT & PN dinamis |
| 17  | Mitigasi Kendala    | Resolusi bottlenecks arsitektur  |
| 18  | Arsitektur Keamanan | Proteksi Web Shell & Dual JWT    |

---

## 🔨 Perintah Berguna

```bash
# Status semua container
docker compose ps

# Lihat log realtime
docker compose logs -f api-server

# Restart Nginx (setelah update config)
docker compose restart nginx

# Update ke versi terbaru
git pull && docker compose up -d --build

# Masuk ke database
docker exec -it pengadilan_db psql -U postgres -d pengadilan_db

# Trigger backup manual
docker exec pengadilan_api bun run backup
```

---

## ♿ Aksesibilitas

Sistem ini memenuhi standar **WCAG 2.1**:

- ✅ Kontras warna minimum 4.5:1
- ✅ Screen reader compatible (alt text, aria-label, semantik HTML)
- ✅ Navigasi keyboard-only
- ✅ Video Juru Bahasa Isyarat untuk layanan PTSP
- ✅ Font scale hingga 200% tanpa merusak layout
- ✅ Touch target minimum 44×44px

---

## 📜 Kepatuhan Regulasi

| Regulasi                                             | Status               |
| ---------------------------------------------------- | -------------------- |
| SK DJU No. 127/DJU/SK.HM1.1/III/2026                 | ✅ Diimplementasikan |
| SK KMA No. 2-144/KMA/SK/VIII/2022 (Informasi Publik) | ✅ Diimplementasikan |
| UU No. 14 Tahun 2008 (Keterbukaan Informasi Publik)  | ✅ Diimplementasikan |
| WCAG 2.1 Aksesibilitas                               | ✅ Diimplementasikan |
| PERMA tentang e-Litigasi & e-Berpadu                 | ✅ Diintegrasikan    |

---

## 👨‍💻 Kontribusi & Pengembangan

Proyek ini dikembangkan untuk lingkungan internal Mahkamah Agung RI.  
Baca [`rules.md`](./rules.md) sebelum berkontribusi — dokumen tersebut adalah panduan wajib.

```bash
# Branch convention
main      → Production (stable)
develop   → Staging
feature/* → Fitur baru

# Commit convention (Conventional Commits)
feat:     → Fitur baru
fix:      → Bug fix
docs:     → Perubahan dokumentasi
refactor: → Refaktor kode
chore:    → Setup & konfigurasi
```

---

<div align="center">

**Dibangun dengan ❤️ untuk pelayanan hukum yang lebih baik di Indonesia**

_Mahkamah Agung Republik Indonesia — Direktorat Jenderal Badan Peradilan Umum_

</div>
