# 03. Frontend Architecture - The Content King (Astro + React + Shadcn/UI)

## Core Technology

- **Framework**: [Astro](https://astro.build/) (Fokus utama pada performa, SSG/SSR hibrida).
- **UI Kit & Komponen**: Terintegrasi mutlak dengan **Tailwind CSS** dan mengadopsi standar emas **Shadcn/UI**. Konstruksi komponen Tabel, Kalender, *Dialog Box* yang responsif dibangun tanpa gesekan besar karena Astro.
- **UI Islands**: Integrasi **React** digunakan khusus untuk merender pemicu *Shadcn components* yang interaktif (seperti Dashboard).
- **Language**: TypeScript (Sangat ketat, membagi tipe dari backend).
- **Styling Dasar**: Tailwind CSS Utility Class dipadukan dengan *CSS Variables* murni (untuk kemudahan theme customizer Admin warna primer Institusi).
- **Data Fetching**: **Elysia Eden Connector** (E2E Type-Safety, memanggil endpoint API seolah memanggil _local function_).

---

## Performance Strategy (The Speed Stack)

- **Zero-JS by Default**: Astro mengonversi semua komponen ke HTML statis. Browser tidak menerima _payload_ JavaScript besar untuk halaman publik statis.
- **Astro Islands Architecture**: Hanya komponen interaktif (Dashboard Admin, Form Pengaduan, Login Publik) yang menerima React JS. Seluruh kerangka halaman publik tetap Zero-JS HTML murni.
- **SSR Mode**: Astro dikonfigurasi dengan adapter `server` (Bun runtime). Request ke Elysia ditarik dari internal network sebelum HTML dikirim ke klien.

---

## Halaman & Rute (Page Map)

### Area Publik (`/`)
```
/                          → Beranda dinamis (Layout Blocks SSR)
/berita                    → Daftar berita (filter: kategori, tag, tipe)
/berita/[slug]             → Detail berita (+ SEO OG tags dari posts table)
/pengumuman                → Daftar pengumuman
/pengumuman/[slug]
/tentang/sejarah           → institution_profiles['sejarah']
/tentang/visi-misi         → institution_profiles['visi'] + ['misi']
/tentang/struktur-organisasi
/tentang/profil-hakim      → staff_members (HAKIM)
/tentang/profil-pegawai    → staff_members (semua division)
/tentang/role-model        → staff_members WHERE is_role_model=true
/tentang/agen-perubahan    → staff_members WHERE is_agen_perubahan=true
/tentang/kepaniteraan      → Dinamis berdasarkan court_class (Tipikor/PHI conditional)
/tentang/kesekretariatan
/layanan/ptsp              → Jenis layanan PTSP + standar pelayanan
/layanan/disabilitas       → service_types WITH sign_language_video_url
/layanan/jadwal-sidang     → Iframe SIPP dari global_configs.sipp_iframe_url
/layanan/pengaduan         → Form online pengaduan (Dynamic Form Builder)
/layanan/informasi-perkara → Link SIPP, Direktori Putusan
/layanan/dip               → Daftar Informasi Publik (DIP)
/layanan/laporan           → Archive laporan publik (filter: type, year)
/layanan/laporan/[id]      → Detail laporan + lampiran (render sesuai display_mode)
/layanan/hukum             → Layanan hukum: Prodeo, Posbakum, Prosedur
/berita/galeri-foto        → galleries (type: PHOTO)
/berita/galeri-video       → galleries (type: VIDEO)
/reformasi-birokrasi/zona-integritas → zi_status + zi_areas + zi_documents
/reformasi-birokrasi/ampuh           → ampuh_certifications + ampuh_areas + ampuh_evidences
/hubungi-kami              → institution_profiles['alamat', 'telp', 'sosmed']
/pencarian                 → Full-text search posts (via Elysia endpoint)
```

### Area Publik Login (`/akun/`)
```
/akun/masuk                → Form login masyarakat (React Island + OTP)
/akun/daftar               → Form registrasi mandiri (NIK + WA OTP)
/akun/dashboard            → Dashboard tiket/pengaduan masyarakat
/akun/tiket/[id]           → Detail tiket + riwayat ticket_replies
/akun/profil               → Edit profil akun publik
```

### Area Admin (`/admin/`)
```
/admin                     → Dashboard ringkasan (stats, berita terbaru, tiket open)
/admin/berita              → Manajemen posts (tabel Shadcn: sort, filter, pagination)
/admin/berita/baru         → Form editor berita (Rich Text + SEO fields)
/admin/berita/[id]/edit
/admin/berita/[id]/review  → Halaman review & approve/reject
/admin/kategori            → CRUD categories (hierarki drag & drop)
/admin/tag                 → CRUD tags
/admin/media               → Library media_assets (uploader + grid)
/admin/laporan             → Manajemen public_reports + attachments
/admin/galeri              → CRUD galleries + gallery_items
/admin/pegawai             → CRUD staff_members (Hakim/Pegawai/Role Model)
/admin/profil-institusi    → Edit institution_profiles key-value
/admin/layanan             → CRUD service_types + sign_language_video
/admin/tiket               → Antrian tiket publik + balas ticket_replies
/admin/form-builder        → CRUD dynamic_forms + form_fields
/admin/form/[id]/data      → Hasil form_submissions + Export Excel/PDF
/admin/layout/homepage     → Drag & Drop layout_blocks order
/admin/navigasi            → Tree editor navigation_menus
/admin/modul-kustom        → CRUD custom_modules (Rich Text / HTML / Script)
/admin/zi                  → Zona Integritas: status WBK/WBBM + 6 area + upload dokumen
/admin/ampuh               → AMPUH: 9 kriteria penilaian + ceklis evidence_items per kriteria + upload bukti
/admin/rb-kegiatan         → CRUD rb_activities + foto
/admin/jaringan-pn         → CRUD sub_courts (khusus mode PT)
/admin/pengguna            → Manajemen users Admin
/admin/pengguna-publik     → Manajemen public_accounts (reset password)
/admin/audit               → audit_logs (filter: user, module, rentang waktu)
/admin/backup              → Trigger manual backup + daftar file backup
/admin/pengaturan          → global_configs (Toggle PT/PN, URL, Warna, Logo)
/admin/statistik           → site_stats harian + visualisasi
```

---

## Key Requirements (Juknis 2026)

- **Aesthetics & Dinamisasi**: Warna tema diinjeksikan via CSS Variables `--primary`, `--secondary` yang dibaca dari `global_configs` di SSR init.
- **Accessibility (WCAG 2.1)**: Semua gambar wajib alt text, screen reader compatible, keyboard navigation, rasio kontras 4.5:1 minimum.
- **Halaman Laporan**: Komponen `<ReportViewer>` (Shadcn/React Island) merender file sesuai `display_mode`:
  - `INLINE_VIEWER` → `<iframe>` PDF viewer full-height
  - `DOWNLOAD_BUTTON` → `<a download>` dengan ikon + ukuran file
  - `PREVIEW_CARD` → Card thumbnail + metadata + CTA
  - `EMBED_LINK` → Link ke situs eksternal target blank
- **AMPUH Dashboard**: Komponen progress bar 7 area dengan warna indikator (merah = belum lengkap, hijau = verified).

---

## Component Handling

1. `.astro` **Files**: Struktur kerangka web (Layout, Header, Footer, Hero Section, halaman statis publik).
2. `.jsx / .tsx` **React Files** (Island): Komponen interaktif saja:
   - `<DragDropLayoutEditor />` — sortable layout blocks homepage
   - `<NavigationTreeEditor />` — tree drag & drop menu builder
   - `<RichTextEditor />` — editor berita (TipTap/Quill)
   - `<ReportViewer />` — renderer laporan multi-mode
   - `<PublicLoginForm />` — form login + OTP masyarakat
   - `<TicketTracker />` — dashboard tiket masyarakat
   - `<AMPUHEvidenceUploader />` — upload & checklist AMPUH
   - `<ZIProgressDashboard />` — progress 6 area ZI
   - `<SiteStatsChart />` — visualisasi statistik pengunjung
   - `<FormBuilder />` — drag & drop form builder Admin
   - `<MediaLibrary />` — grid media_assets dengan search
   - `<MobileNav />` — hamburger + drawer navigasi (aktif hanya di < md breakpoint)
   - `<HeroSlider />` — carousel banner dengan swipe gesture + dots pagination untuk mobile

