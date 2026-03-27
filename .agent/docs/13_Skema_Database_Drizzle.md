# 13. Skema Basis Data Drizzle (The Speed Stack Brain)

Sebagai panduan pemrogram (`api-server`), desain tabel database Postgres (via Drizzle ORM) ini mutlak dipedomani agar mampu menangani sistem tersentralisasi Pengadilan Tinggi (PT) dan Pengadilan Negeri (PN) beserta login masyarakat.

---

## 1. Domain Pengguna & Otorisasi (`users`, `public_accounts`)

```
enum USER_ROLE = ['SUPERADMIN', 'EDITOR', 'ADMIN_LAYANAN', 'PUBLIC']
```

**`users` Table** (Internal Pegawai / Admin)
- `id` (uuid, pk)
- `nip` (varchar, unique)
- `name` (varchar)
- `email` (varchar, unique)
- `password_hash` (varchar) → Argon2
- `role` (enum USER_ROLE)
- `is_active` (boolean, default true)
- `last_login` (timestamp)
- `created_at` (timestamp)

**`public_accounts` Table** (Masyarakat Umum)
- `id` (uuid, pk)
- `nik` (varchar, 16, unique)
- `name` (varchar)
- `whatsapp_number` (varchar) → OTP / Notif Tiket
- `email` (varchar, unique)
- `password_hash` (varchar)
- `is_verified` (boolean, default false)
- `created_at` (timestamp)

---

## 2. Domain Keamanan (`audit_logs`)

**`audit_logs` Table**
- `id` (uuid, pk)
- `user_id` (uuid, ref: users.id, nullable) → null jika aksi sistem otomatis
- `action` (varchar) → e.g. `UPDATE_GLOBAL_CONFIG`, `DELETE_NEWS`, `PUBLISH_POST`
- `module` (varchar) → Modul yang terpengaruh (e.g. `posts`, `global_configs`)
- `ip_address` (varchar)
- `payload_before` (jsonb) → Snapshot data sebelum perubahan
- `payload_after` (jsonb) → Snapshot data sesudah perubahan
- `timestamp` (timestamp)

---

## 3. Domain Konfigurasi Global (`global_configs`)

Singleton (1 baris saja). Dibaca oleh Astro di SSR init untuk menentukan identitas dan perilaku seluruh sistem.

**`global_configs` Table**
- `id` (int, pk = 1)
- `tenant_mode` (enum: `PENGADILAN_TINGGI`, `PENGADILAN_NEGERI`)
- `court_name` (varchar) → e.g. "Pengadilan Negeri Surabaya"
- `court_class` (enum: `IA_KHUSUS`, `IA`, `IB`, `II`) → aktif/tidaknya Kepaniteraan Tipikor, PHI, dll
- `logo_url` (varchar) → Path logo unggahan
- `favicon_url` (varchar)
- `wbk_active` (boolean) → Badge WBK tampil atau tidak
- `wbbm_active` (boolean) → Badge WBBM tampil atau tidak
- `primary_color` (varchar, default `#9a2109`)
- `secondary_color` (varchar)
- `sipp_iframe_url` (varchar) → URL iframe jadwal sidang dari SIPP
- `e_court_url` (varchar)
- `e_berpadu_url` (varchar)
- `siwas_url` (varchar)
- `sisuper_url` (varchar)
- `direktori_putusan_url` (varchar)
- `jdih_url` (varchar)
- `lpse_url` (varchar)
- `ma_ri_url` (varchar) → Link ke Mahkamah Agung
- `parent_court_name` (varchar) → Nama PT Pembina (jika mode PN)
- `parent_court_url` (varchar) → URL PT Pembina (jika mode PN)
- `maintenance_mode` (boolean, default false)
- `maintenance_message` (text)
- `analytics_script` (text, nullable) → Script tracker (Matomo/Umami)
- `footer_script` (text, nullable) → Script injeksi footer kustom
- `updated_at` (timestamp)

---

## 4. Domain CMS Berita & Konten (`categories`, `tags`, `posts`, `post_tags`, `post_revisions`, `media_assets`)

**`categories` Table** (Mendukung Hierarki Berjenjang)
- `id` (serial, pk)
- `parent_id` (int, ref: categories.id, nullable) → Self-referencing untuk sub-kategori (e.g. Berita > Pidana)
- `name` (varchar, unique)
- `slug` (varchar, unique)
- `description` (text, nullable)
- `order_index` (int, default 0)
- `is_active` (boolean, default true)

**`tags` Table**
- `id` (serial, pk)
- `name` (varchar, unique) → e.g. "tipikor", "ptsp", "reforma birokrasi"
- `slug` (varchar, unique)

**`post_tags` Table** (Relasi Many-to-Many: posts ↔ tags)
- `post_id` (int, ref: posts.id)
- `tag_id` (int, ref: tags.id)
- *Primary Key: composite (post_id, tag_id)*

**`posts` Table** (Berita, Pengumuman, Artikel — sistem lengkap blog modern)
- `id` (serial, pk)
- `post_type` (enum: `BERITA`, `PENGUMUMAN`, `ARTIKEL`, `SIARAN_PERS`) → Membedakan jenis konten
- `category_id` (int, ref: categories.id)
- `slug` (varchar, unique)
- `lang` (enum: `ID`, `EN`, default: `ID`) → Dukungan Language Switcher
- `title` (varchar)
- `excerpt` (text, nullable) → Ringkasan 150 karakter untuk card & meta description fallback
- `html_content` (text) → Konten penuh dari editor Rich Text / WYSIWYG
- `thumbnail_url` (varchar, nullable) → URL gambar utama artikel
- `thumbnail_alt` (varchar, nullable) → Alt text thumbnail (wajib WCAG 2.1)
- `reading_time_minutes` (int, nullable) → Estimasi waktu baca (auto-hitung dari panjang konten)
- `author_id` (uuid, ref: users.id)
- `reviewer_id` (uuid, ref: users.id, nullable) → Editor yang me-review sebelum publish
- `approved_by` (uuid, ref: users.id, nullable) → Super Admin/PPID yang memberikan persetujuan akhir
- `status` (enum: `DRAFT`, `IN_REVIEW`, `PUBLISHED`, `REJECTED`, `ARCHIVED`)
- `rejection_reason` (text, nullable) → Catatan penolakan dari reviewer ke penulis
- `is_featured` (boolean, default false) → Tampil di posisi prioritas/utama beranda
- `is_pinned` (boolean, default false) → Disematkan di atas daftar berita
- `allow_comments` (boolean, default false) → Toggle kolom komentar publik
- `view_count` (int, default 0) → Total pembacaan
- `share_count` (int, default 0) → Total berbagi (diupdate via API endpoint share)
- `meta_title` (varchar, nullable) → SEO: Title tag spesifik (maks 60 karakter)
- `meta_description` (varchar, nullable) → SEO: Meta description (maks 160 karakter)
- `og_image_url` (varchar, nullable) → SEO: Gambar Open Graph (preview WhatsApp/sosmed)
- `og_image_alt` (varchar, nullable) → Alt text Open Graph
- `canonical_url` (varchar, nullable) → Jika konten ini adalah re-publish dari sumber lain
- `published_at` (timestamp, nullable)
- `created_at` (timestamp)
- `updated_at` (timestamp)

**`post_revisions` Table** (Riwayat Perubahan Konten — Audit Versi)
- `id` (serial, pk)
- `post_id` (int, ref: posts.id)
- `editor_id` (uuid, ref: users.id) → Siapa yang melakukan perubahan
- `title_snapshot` (varchar) → Judul pada saat revisi dilakukan
- `content_snapshot` (text) → Isi konten pada saat revisi dilakukan
- `revision_note` (varchar, nullable) → Catatan singkat perubahan
- `created_at` (timestamp)

**`media_assets` Table** (Pustaka Media Terpusat)
- `id` (uuid, pk)
- `uploader_id` (uuid, ref: users.id)
- `file_name` (varchar) → Nama file asli
- `stored_name` (varchar) → Nama file yang disimpan di server (UUID-based, aman)
- `file_url` (varchar) → Path statis di `/public/uploads/`
- `mime_type` (varchar) → e.g. `image/webp`, `application/pdf`, `video/mp4`
- `file_size_bytes` (int)
- `width` (int, nullable) → Dimensi gambar px (untuk lazy loading hints)
- `height` (int, nullable)
- `alt_text` (varchar, nullable) → Wajib diisi untuk aksesibilitas WCAG 2.1
- `caption` (text, nullable) → Keterangan gambar
- `created_at` (timestamp)

---

## 5. Domain Laporan Publik & Manajemen File (`public_reports`, `report_attachments`)

Domain ini mengelola seluruh dokumen/laporan resmi yang wajib dipublikasikan sesuai Juknis SK KMA 2-144/2022 dan Juknis DJU 127/2026.

```
enum REPORT_TYPE = [
  -- Keuangan & Anggaran --
  'DIPA',             -- Daftar Isian Pelaksanaan Anggaran
  'RKAKL',            -- Rencana Kerja dan Anggaran K/L
  'REALISASI_ANGGARAN', -- Laporan Realisasi Anggaran
  'LAPORAN_KEUANGAN', -- Laporan Keuangan (diaudit BPK)
  'SISA_PANJAR',      -- Informasi Sisa Panjar Perkara
  -- Kinerja & Akuntabilitas --
  'LKJIP',            -- Laporan Kinerja Instansi Pemerintah
  'SAKIP',            -- Sistem Akuntabilitas Kinerja IP
  'LAPORAN_TAHUNAN',  -- Laporan Tahunan Pengadilan
  'HASIL_PENELITIAN', -- Publikasi hasil penelitian internal
  -- Kepegawaian & Integritas --
  'LHKPN',            -- Laporan Harta Kekayaan Pejabat Negara
  'SPT_TAHUNAN',      -- Laporan SPT Tahunan Pegawai
  -- Aset --
  'DAFTAR_ASET',      -- Ringkasan Daftar Aset dan Inventaris
  -- Pelayanan Publik --
  'DIP',              -- Daftar Informasi Publik
  'LAYANAN_INFORMASI', -- Laporan Layanan Informasi Publik
  'SKM',              -- Survei Kepuasan Masyarakat
  'SPAK',             -- Survei Persepsi Anti Korupsi
  'IKM_TRIWULAN',     -- Indeks Kepuasan Masyarakat per Triwulan
  'IPAK_TRIWULAN',    -- Indeks Persepsi Anti Korupsi per Triwulan
  'SURVEI_HARIAN',    -- Laporan Survei Harian
  -- Inovasi & Sosialisasi --
  'E_BROSUR',         -- Brosur elektronik layanan pengadilan
  'INOVASI',          -- Dokumentasi inovasi satker
  -- Lainnya --
  'LAIN_LAIN'
]

enum DISPLAY_MODE = [
  'INLINE_VIEWER',   -- PDF dibuka langsung di halaman (iframe PDF viewer)
  'DOWNLOAD_BUTTON', -- Tombol unduh langsung
  'PREVIEW_CARD',    -- Card deskriptif dengan link eksternal
  'EMBED_LINK'       -- Embed URL eksternal (misalnya laporan di situs MA)
]

enum REPORT_PERIOD = ['TAHUNAN', 'SEMESTERAN', 'TRIWULAN', 'BULANAN', 'INSIDENTAL']
```

**`public_reports` Table**
- `id` (serial, pk)
- `report_type` (enum REPORT_TYPE)
- `title` (varchar) → Judul resmi laporan
- `description` (text, nullable) → Narasi/ringkasan laporan untuk tampil di halaman
- `year` (int) → Tahun laporan
- `period_type` (enum REPORT_PERIOD) → Frekuensi laporan
- `period_value` (int, nullable) → Nilai periode: semester (1/2), triwulan (1/2/3/4), bulan (1-12)
- `display_mode` (enum DISPLAY_MODE, default `DOWNLOAD_BUTTON`) → **Cara menampilkan file di frontend**
- `main_file_url` (varchar, nullable) → Path file utama di `/public/uploads/` (jika upload lokal)
- `external_url` (varchar, nullable) → URL eksternal jika file ada di situs MA/BPK/KPK
- `thumbnail_preview_url` (varchar, nullable) → Gambar preview halaman pertama PDF (opsional)
- `file_size_bytes` (int, nullable)
- `page_slugs` (jsonb, nullable) → Array halaman yang menampilkan laporan ini
  → e.g. `["/layanan/laporan", "/tentang/keuangan"]`
- `target_section` (varchar, nullable) → Section/anchor di halaman tujuan
- `is_active` (boolean, default true) → Toggle publikasi (false = draft/tersembunyi)
- `is_featured` (boolean, default false) → Tampil di widget sidebar atau homepage
- `order_index` (int, default 0) → Urutan tampil dalam daftar
- `uploaded_by` (uuid, ref: users.id)
- `published_at` (timestamp, nullable)
- `created_at` (timestamp)
- `updated_at` (timestamp)

**`report_attachments` Table** (Lampiran/File Pendukung Ganda per Laporan)
- `id` (serial, pk)
- `report_id` (int, ref: public_reports.id)
- `attachment_label` (varchar) → e.g. "Lampiran I", "Buku II Neraca", "Ringkasan Eksekutif"
- `file_url` (varchar)
- `file_name` (varchar)
- `file_size_bytes` (int)
- `display_mode` (enum DISPLAY_MODE, default `DOWNLOAD_BUTTON`) → Override cara tampil per lampiran
- `order_index` (int, default 0)
- `created_at` (timestamp)

---

> **Catatan Implementasi Frontend (Astro):**
> - `INLINE_VIEWER` → merender `<iframe src="{file_url}" />` atau komponen `<PDFViewer>` (Shadcn/React)
> - `DOWNLOAD_BUTTON` → merender `<a href="{file_url}" download>` dengan ikon unduh
> - `PREVIEW_CARD` → merender card dengan `thumbnail_preview_url` + judul + link
> - `EMBED_LINK` → merender `<a href="{external_url}" target="_blank">` ke situs luar

---

## 6. Domain Profil Institusi (`institution_profiles`, `staff_members`)

Menampung konten statis CMS untuk halaman "Tentang Pengadilan" yang wajib ada sesuai Juknis.

**`institution_profiles` Table** (Singleton per Keys)
- `id` (serial, pk)
- `key` (varchar, unique) → Misal: `visi`, `misi`, `sejarah`, `tupoksi`, `sambutan_ketua`, `wilayah_yurisdiksi`, `denah_lokasi_url`, `maps_embed_url`
- `value_text` (text) → Konten teks/HTML ringan
- `updated_at` (timestamp)

**`staff_members` Table** (Hakim, Pegawai, Role Model, Agen Perubahan)
- `id` (serial, pk)
- `name` (varchar)
- `nip` (varchar, nullable)
- `position` (varchar) → Jabatan e.g. "Ketua Pengadilan", "Hakim", "Panitera Pengganti"
- `division` (enum: `HAKIM`, `KEPANITERAAN_PIDANA`, `KEPANITERAAN_PERDATA`, `KEPANITERAAN_HUKUM`, `KEPANITERAAN_TIPIKOR`, `KEPANITERAAN_PHI`, `KEPANITERAAN_NIAGA`, `KEPANITERAAN_PERIKANAN`, `KESEKRETARIATAN`, `JURUSITA`)
- `photo_url` (varchar, nullable)
- `education_history` (jsonb, nullable) → Array riwayat pendidikan
- `career_history` (jsonb, nullable) → Array riwayat jabatan
- `is_role_model` (boolean, default false) → Tandai sebagai Role Model Ketua
- `is_agen_perubahan` (boolean, default false) → Tandai sebagai Agen Perubahan
- `commitment_text` (text, nullable) → Narasi komitmen (Role Model/Agen Perubahan)
- `is_active` (boolean, default true)
- `order_index` (int, default 0) → Urutan tampil
- `created_at` (timestamp)

---

## 7. Domain Layanan Masyarakat & PTSP (`service_types`, `public_tickets`, `ticket_replies`)

**`service_types` Table**
- `id` (serial, pk)
- `name` (varchar) → e.g. "Layanan Perdata", "Layanan Pidana"
- `description` (text, nullable)
- `sign_language_video_url` (varchar, nullable) → Video Juru Bahasa Isyarat
- `standard_service_doc_url` (varchar, nullable) → PDF Standar Pelayanan
- `is_active` (boolean, default true)
- `order_index` (int)

**`public_tickets` Table** (Pengaduan & Permohonan Informasi)
- `id` (uuid, pk)
- `public_account_id` (uuid, ref: public_accounts.id)
- `assigned_to` (uuid, ref: users.id, nullable) → Admin Layanan yang menangani
- `ticket_number` (varchar, unique) → Auto-generate e.g. `TKT-2026-0001`
- `subject` (varchar)
- `ticket_type` (enum: `PENGADUAN`, `PERMOHONAN_INFORMASI`, `SARAN`, `SURVEY`)
- `status` (enum: `OPEN`, `IN_PROGRESS`, `WAITING_RESPONSE`, `CLOSED`, `REJECTED`)
- `priority` (enum: `LOW`, `NORMAL`, `HIGH`)
- `json_payload` (jsonb) → Isi form dinamis dari Form Builder
- `closed_at` (timestamp, nullable)
- `created_at` (timestamp)

**`ticket_replies` Table** (History balasan tiket)
- `id` (serial, pk)
- `ticket_id` (uuid, ref: public_tickets.id)
- `sender_type` (enum: `PUBLIC`, `ADMIN`)
- `sender_id` (uuid) → ID publik atau admin
- `message` (text)
- `attachment_url` (varchar, nullable)
- `created_at` (timestamp)

---

## 8. Domain Form Builder Dinamis (`dynamic_forms`, `form_fields`, `form_submissions`)

Mendukung Admin membuat form online (Survey IKM, IPAK, Pengaduan Khusus) tanpa coding.

**`dynamic_forms` Table**
- `id` (serial, pk)
- `title` (varchar) → e.g. "Survey IKM Q1 2026"
- `description` (text, nullable)
- `form_type` (enum: `SURVEY_IKM`, `SURVEY_IPAK`, `PENGADUAN`, `UMUM`)
- `is_active` (boolean, default true)
- `requires_login` (boolean, default false) → Hanya bisa diisi akun publik terverifikasi
- `created_by` (uuid, ref: users.id)
- `created_at` (timestamp)

**`form_fields` Table**
- `id` (serial, pk)
- `form_id` (int, ref: dynamic_forms.id)
- `field_type` (enum: `TEXT`, `TEXTAREA`, `SELECT`, `RADIO`, `CHECKBOX`, `FILE`, `RATING`)
- `label` (varchar)
- `placeholder` (varchar, nullable)
- `options` (jsonb, nullable) → Pilihan untuk SELECT/RADIO/CHECKBOX
- `is_required` (boolean, default false)
- `order_index` (int)

**`form_submissions` Table**
- `id` (uuid, pk)
- `form_id` (int, ref: dynamic_forms.id)
- `public_account_id` (uuid, ref: public_accounts.id, nullable) → Nullable jika anonymous
- `answers` (jsonb) → `{"field_id": "jawaban", ...}`
- `submitted_at` (timestamp)

---

## 9. Domain Tata Letak Dinamis (`layout_blocks`, `navigation_menus`, `hero_slides`)

**`hero_slides` Table** (Manajemen Slide Banner)
- `id` (serial, pk)
- `image_url` (varchar)
- `title` (varchar, nullable) → Overlay judul
- `subtitle` (varchar, nullable)
- `link_url` (varchar, nullable) → CTA klik slide
- `order_index` (int)
- `is_active` (boolean, default true)

**`layout_blocks` Table** (Dinamika Blok Homepage)
- `id` (serial, pk)
- `block_type` (enum: `HERO_SLIDER`, `QUICK_LINKS`, `NEWS_GRID`, `ANNOUNCEMENT_LIST`, `SIPP_WIDGET`, `RSS_FEED`, `VIDEO_BANNER`, `CUSTOM_MODULE`, `MAKLUMAT_PELAYANAN`)
- `is_active` (boolean, default true)
- `order_index` (int)
- `config_payload` (jsonb) → Parameter visual spesifik (limit berita, URL RSS, dsb.)

**`navigation_menus` Table** (Menu Bertingkat)
- `id` (serial, pk)
- `parent_id` (int, ref: navigation_menus.id, nullable) → Self-referencing untuk dropdown
- `label` (varchar)
- `url` (varchar) → Internal path atau URL eksternal
- `icon_name` (varchar, nullable) → Nama ikon Lucide/Heroicon
- `order_index` (int)
- `is_active` (boolean, default true)
- `visibility` (enum: `ALL`, `PT_ONLY`, `PN_ONLY`)
- `open_in_new_tab` (boolean, default false)

---

## 10. Domain Modul Kustom (`custom_modules`)

**`custom_modules` Table** (Blok Bebas & Integrasi Legacy PHP)
- `id` (serial, pk)
- `title` (varchar) → Nama internal misal: "Banner HUT Pengadilan"
- `content_type` (enum: `RICH_TEXT`, `RAW_HTML`, `EXTERNAL_SCRIPT`) → `EXTERNAL_SCRIPT` memungkinkan JS fetch ke endpoint PHP legacy
- `content_body` (text) → Konten WYSIWYG, HTML, atau tag `<script>` penarik PHP endpoint
- `display_scope` (enum: `ALL_PAGES`, `SPECIFIC_PAGES`, `HOMEPAGE_ONLY`)
- `target_urls` (jsonb, nullable) → Array path jika `SPECIFIC_PAGES`
- `ui_position` (enum: `TOP_BANNER`, `BOTTOM_BANNER`, `SIDEBAR`, `ABOVE_FOOTER`, `CONTENT_AREA`)
- `is_active` (boolean, default true)

---

## 11. Domain Jaringan Sub-Pengadilan (`sub_courts`)

Khusus digunakan saat Mode = `PENGADILAN_TINGGI`. Menyimpan daftar PN di bawah wilayah hukum PT untuk ditampilkan di beranda dan sidebar.

**`sub_courts` Table**
- `id` (serial, pk)
- `name` (varchar) → e.g. "Pengadilan Negeri Surabaya"
- `court_class` (enum: `IA_KHUSUS`, `IA`, `IB`, `II`)
- `website_url` (varchar)
- `city` (varchar)
- `province` (varchar)
- `is_active` (boolean, default true)
- `order_index` (int)

---

## 12. Domain Statistik Pengunjung (`site_stats`)

**`site_stats` Table** (Snapshot Harian Statistik)
- `id` (serial, pk)
- `date` (date, unique)
- `total_visitors` (int, default 0) → Akumulasi unik IP
- `daily_visitors` (int, default 0)
- `page_views` (int, default 0)
- `top_pages` (jsonb) → Array halaman terpopuler hari itu

---

## 13. Domain Reformasi Birokrasi (`zi_areas`, `zi_documents`, `ampuh_certifications`, `rb_activities`)

Menunggu kewajiban Juknis DJU 127/2026 pasal Reformasi Birokrasi. Menu ini memiliki dua sub-program besar:

### A. Zona Integritas (ZI — WBK & WBBM)

Zona Integritas dinilai berdasarkan **6 Area Pembangunan** yang masing-masing memiliki dokumen bukti dukung.

```
enum ZI_AREA = [
  'MANAJEMEN_PERUBAHAN',          -- Area 1
  'PENATAAN_TATALAKSANA',         -- Area 2
  'PENATAAN_SISTEM_SDM',          -- Area 3
  'PENGUATAN_AKUNTABILITAS',      -- Area 4
  'PENGUATAN_PENGAWASAN',         -- Area 5
  'PENINGKATAN_LAYANAN_PUBLIK'    -- Area 6
]

enum ZI_PREDIKAT = ['MENUJU_WBK', 'WBK', 'MENUJU_WBBM', 'WBBM']
```

**`zi_status` Table** (Status dan Riwayat Predikat ZI — Singleton/Timeline)
- `id` (serial, pk)
- `predikat` (enum ZI_PREDIKAT) → Predikat yang sedang diraih/dimiliki
- `year` (int) → Tahun penilaian
- `assessment_score` (decimal, nullable) → Nilai akhir penilaian
- `assessor_institution` (varchar, nullable) → e.g. "KemenPANRB", "Ombudsman"
- `certificate_url` (varchar, nullable) → File SK/Sertifikat predikat
- `lke_url` (varchar, nullable) → Link LKE Zona Integritas (portal penilaian)
- `is_current` (boolean, default true) → Status terkini vs historis
- `created_at` (timestamp)

**`zi_areas` Table** (6 Area Kerja Zona Integritas)
- `id` (serial, pk)
- `area` (enum ZI_AREA, unique)
- `area_label` (varchar) → Label tampil misalnya "Area 1: Manajemen Perubahan"
- `description` (text, nullable) → Narasi singkat area ini
- `progress_percent` (int, default 0) → Persentase progres (0-100)
- `order_index` (int)

**`zi_documents` Table** (Dokumen Bukti Dukung per Area ZI)
- `id` (serial, pk)
- `zi_area_id` (int, ref: zi_areas.id)
- `document_name` (varchar) → e.g. "SK Tim Pokja Zona Integritas"
- `document_category` (varchar) → e.g. "Perencanaan", "Monitoring", "Evaluasi"
- `file_url` (varchar, nullable) → PDF bukti dukung
- `external_url` (varchar, nullable) → Link jika dokumen ada di sistem luar
- `display_mode` (enum DISPLAY_MODE, default `DOWNLOAD_BUTTON`)
- `is_public` (boolean, default true) → Apakah dokumen bisa diakses publik
- `year` (int)
- `order_index` (int)
- `uploaded_by` (uuid, ref: users.id)
- `created_at` (timestamp)

### B. AMPUH — Akreditasi Penjaminan Mutu Pengadilan Unggul dan Tangguh

Berdasarkan riset sumber resmi Mahkamah Agung RI, AMPUH Badilum menggunakan istilah **"Kriteria"** (bukan Area). Terdapat **9 Kriteria Penilaian** resmi.

```
enum AMPUH_LEVEL  = ['PARIPURNA', 'UNGGUL']  -- Predikat resmi sertifikat
enum AMPUH_STATUS = ['DALAM_PROSES', 'TERAKREDITASI', 'KADALUARSA', 'DICABUT']

enum AMPUH_KRITERIA = [
  'MANAJEMEN_PERADILAN',          -- Kriteria 1
  'ADMINISTRASI_PERKARA',         -- Kriteria 2
  'ADMINISTRASI_PERSIDANGAN',     -- Kriteria 3
  'ADMINISTRASI_UMUM',            -- Kriteria 4
  'PELAYANAN_PUBLIK',             -- Kriteria 5
  'PENGELOLAAN_KEUANGAN',         -- Kriteria 6
  'PENGADAAN_BARANG_JASA',        -- Kriteria 7
  'PENGAWASAN',                   -- Kriteria 8
  'PENANGANAN_PENGADUAN'          -- Kriteria 9
]
```

**`ampuh_certifications` Table** (Sertifikasi Mutu AMPUH dari Badilum)
- `id` (serial, pk)
- `certification_level` (enum AMPUH_LEVEL) → Predikat: Paripurna / Unggul
- `status` (enum AMPUH_STATUS)
- `year` (int) → Tahun akreditasi
- `certificate_number` (varchar, nullable) → Nomor sertifikat resmi Badilum
- `certificate_url` (varchar, nullable) → File PDF sertifikat AMPUH
- `valid_from` (date, nullable)
- `valid_until` (date, nullable) → Masa berlaku sertifikat
- `assessment_score` (decimal, nullable) → Nilai total asesmen
- `assessor_name` (varchar, nullable) → Nama asesor dari Badilum/PT
- `assessment_report_url` (varchar, nullable) → Laporan hasil asesmen
- `is_current` (boolean, default true)
- `created_at` (timestamp)

**`ampuh_kriteria` Table** (9 Kriteria Penilaian AMPUH — bukan "area")
- `id` (serial, pk)
- `kriteria` (enum AMPUH_KRITERIA, unique)
- `kriteria_label` (varchar) → e.g. "Kriteria 1: Manajemen Peradilan"
- `description` (text) → Penjelasan kriteria
- `max_score` (decimal) → Bobot skor maksimal kriteria ini
- `achieved_score` (decimal, nullable) → Nilai terkini yang dicapai
- `order_index` (int)

**`ampuh_evidence_items` Table** (Daftar Item Evidence/Bukti per Kriteria — Inti Ceklis AMPUH)
- `id` (serial, pk)
- `ampuh_kriteria_id` (int, ref: ampuh_kriteria.id)
- `item_code` (varchar) → Kode item sesuai ceklis resmi Badilum e.g. "K1.1", "K3.2"
- `item_label` (varchar) → Deskripsi item bukti e.g. "Laporan Hasil Pengawasan dan Pembinaan"
- `item_subcategory` (varchar, nullable) → Sub-kategori dalam kriteria
- `evidence_type` (enum: `DOKUMEN`, `LAPORAN`, `FOTO`, `VIDEO`, `DATA_SISTEM`, `SURVEI`, `LINK_EKSTERNAL`)
- `is_mandatory` (boolean, default true) → Wajib atau opsional
- `description` (text, nullable) → Penjelasan bukti yang dibutuhkan
- `order_index` (int)

**`ampuh_evidences` Table** (Unggahan Bukti Aktual per Item)
- `id` (serial, pk)
- `evidence_item_id` (int, ref: ampuh_evidence_items.id)
- `certification_id` (int, ref: ampuh_certifications.id) → Terkait sesi asesmen mana
- `title` (varchar) → Judul/nama bukti yang diunggah
- `description` (text, nullable)
- `file_url` (varchar, nullable) → File PDF/Foto yang diunggah
- `external_url` (varchar, nullable) → Link ke sistem SIPP/e-Berpadu/SISUPER sebagai bukti
- `display_mode` (enum DISPLAY_MODE)
- `is_verified` (boolean, default false) → Sudah diverifikasi asesor
- `verified_by` (uuid, ref: users.id, nullable)
- `verified_at` (timestamp, nullable)
- `uploaded_by` (uuid, ref: users.id)
- `created_at` (timestamp)

> **Contoh Evidence Item per Kriteria (dari Ceklis AMPUH Badilum):**
> - **K1 Manajemen Peradilan**: Laporan pengawasan & pembinaan, notulen monitoring & evaluasi, bukti tindak lanjut
> - **K2 Administrasi Perkara**: Akurasi data SIPP (tilang, denda, anonimisasi), register perkara, laporan sisa perkara
> - **K3 Administrasi Persidangan**: Berita acara sidang lengkap, bukti e-Litigasi, jadwal sidang terpublikasi
> - **K4 Administrasi Umum**: Dokumen kepegawaian, SKP/evaluasi kinerja, bukti nilai Ber-AKHLAK
> - **K5 Pelayanan Publik**: Laporan SKM & SPAK (SISUPER), bukti layanan PTSP, maklumat pelayanan
> - **K6 Pengelolaan Keuangan**: Laporan keuangan, biaya perkara, laporan realisasi anggaran
> - **K7 Pengadaan Barang & Jasa**: Dokumen lelang LPSE, kontrak pengadaan
> - **K8 Pengawasan**: Bukti sosialisasi PERMA & SK KMA, laporan temuan & tindak lanjut
> - **K9 Penanganan Pengaduan**: Rekap pengaduan (SIWAS/SP4N-LAPOR), bukti tindak lanjut pengaduan, alur pengaduan terpublikasi


### C. Kegiatan Reformasi Birokrasi Umum (`rb_activities`)

Untuk menampilkan kegiatan pendukung ZI dan AMPUH (foto, dokumentasi, narasi kegiatan).

**`rb_activities` Table**
- `id` (serial, pk)
- `activity_type` (enum: `ZONA_INTEGRITAS`, `AMPUH`, `UMUM`)
- `title` (varchar) → Judul kegiatan
- `description` (text, nullable)
- `activity_date` (date)
- `is_published` (boolean, default true)
- `created_by` (uuid, ref: users.id)
- `created_at` (timestamp)

**`rb_activity_photos` Table** (Galeri Foto per Kegiatan RB)
- `id` (serial, pk)
- `activity_id` (int, ref: rb_activities.id)
- `image_url` (varchar)
- `caption` (varchar, nullable)
- `alt_text` (varchar, nullable) → Wajib WCAG
- `order_index` (int)

---

## 14. Domain Galeri Media (`galleries`, `gallery_items`)

Mengelola Foto Galeri dan Video Galeri sesuai Juknis menu Berita (baris 1716-1723).

```
enum GALLERY_TYPE = [
  'KEGIATAN_PENGADILAN',      -- Foto kegiatan resmi
  'FASILITAS_PUBLIK',         -- Foto gedung dan fasilitas
  'SARANA_PERSIDANGAN_ANAK',  -- Ruang sidang anak
  'ZONA_INTEGRITAS',          -- Dokumentasi kegiatan ZI
  'VIDEO_PROFIL',             -- Video profil pengadilan (SE Dirjen 7/2021)
  'VIDEO_PTSP',               -- Video layanan PTSP
  'VIDEO_UMUM'                -- Video lainnya
]
```

**`galleries` Table** (Album/Koleksi)
- `id` (serial, pk)
- `gallery_type` (enum GALLERY_TYPE)
- `title` (varchar) → Judul album e.g. "Peresmian Gedung Baru 2026"
- `description` (text, nullable)
- `cover_image_url` (varchar, nullable) → Cover/thumbnail album
- `is_published` (boolean, default true)
- `created_by` (uuid, ref: users.id)
- `created_at` (timestamp)

**`gallery_items` Table** (Item foto atau video dalam album)
- `id` (serial, pk)
- `gallery_id` (int, ref: galleries.id)
- `media_type` (enum: `PHOTO`, `VIDEO`)
- `file_url` (varchar, nullable) → Path lokal untuk foto
- `video_embed_url` (varchar, nullable) → URL YouTube/embed untuk video
- `thumbnail_url` (varchar, nullable) → Thumbnail video
- `caption` (varchar, nullable) → Keterangan foto sesuai Juknis (wajib 1-5 foto)
- `alt_text` (varchar, nullable) → Wajib WCAG
- `is_sign_language` (boolean, default false) → Tandai video yang menggunakan Juru Bahasa Isyarat
- `order_index` (int)
- `created_at` (timestamp)

---

*Catatan Bagi Agen/Developer: Seluruh tabel di atas adalah referensi mutlak untuk membuat file `apps/api-server/src/db/schema.ts` menggunakan Drizzle ORM. Gunakan `pgTable`, `pgEnum`, dan relasi `relations()` dari `drizzle-orm/pg-core`. Total: 14 Domain, 40+ Tabel.*
