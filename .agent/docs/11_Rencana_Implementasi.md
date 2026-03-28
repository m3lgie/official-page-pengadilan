# 11. Roadmap Implementasi Detail - The Speed Stack

Rencana eksekusi teknis langkah demi langkah untuk membangun Sistem Website Pengadilan tersentralisasi (PT & PN) menggunakan _Astro, Bun, Elysia, Drizzle, PostgreSQL, dan Redis_.

Ini adalah buku petunjuk kerja berbutir presisi untuk mengkonversi spesifikasi _Juknis 2026_ menjadi _codebase_ tervalidasi.

---

## Tahap 1: Inisialisasi Arsitektur Dasar (Monorepo Setup & Database)

**Fokus:** Menyiapkan kerangka kerja _Monorepo_, _runtime Bun_, dan menghubungkan sistem ke basis data _PostgreSQL_.

- **Langkah 1.1:** Setup _Bun Workspace_ di root `package.json` (`/apps/api-server`, `/apps/web-portal`, `/packages/shared-types`).
- **Langkah 1.2:** Konfigurasi Docker (Nginx, PostgreSQL, Redis, Pg-Backup) via `docker-compose.yml`. Pembuatan _Vhost_ awal dan _SSL Self-signed fallback_.
- **Langkah 1.3:** Setup `apps/api-server` dengan _ElysiaJS_ dan konfigurasi _CORS_, WAF/Rate Limiting dasar.
- **Langkah 1.4:** Desain & sinkronisasi Skema Database menggunakan _Drizzle ORM_ (14 Domain: Pimpinan, Layanan, ZI, AMPUH, dsb).
- **Langkah 1.5:** Pembuatan Seeder awal untuk `global_configs` (Identitas Master), struktur kelas peradilan PN tipikor/pidana, dan pembuatan akun asali _Super Admin_.

---

## Tahap 2: Pembangunan Mesin Backend & API Service (Elysia + Drizzle)

**Fokus:** Menghidupkan rute layanan API otentik untuk melayani _Frontend Astro_ dan _Dashboard Admin_.

- **Langkah 2.1:** Implementasi fungsi Autentikasi JWT (Pemecahan rahasia ganda: Admin vs Publik) dengan enkripsi sandi tangguh _Argon2id_.
- **Langkah 2.2:** Pengekangan _TypeBox_ mutlak pada skema muatan input agar tertutup dari anomali variabel sisipan (Cegah XSS/SSRF).
- **Langkah 2.3:** Membangun titik akhir (Endpoints) CRUD Dinamis (Manajemen Profil Institusi, Berita/Pengumuman, Lampiran PDF).
- **Langkah 2.4:** Pembangunan _Role-Based Access Control_ (RBAC) pada lapis Middleware untuk blokade akses lintas-kewenangan antar Admin, Editor, PPID, Staf TI.
- **Langkah 2.5:** Setup manajemen log tak-terbantahkan (`audit_logs`) pada _Payload Before_ dan _Payload After_.
- **Langkah 2.6:** Mengaktifkan _Redis Cache Engine_ untuk memutar kecepatan respons instan pada _Global Configs_ dan Daftar Layanan Statis.
- **Langkah 2.7:** Mengaitkan rute _Media Assets_ dengan detektor paksaan pengaman filter ekstensi dan MIME tipe fail (_No Web Shell/Executable_).

---

## Tahap 3: Konstruksi Frontend Web Publik (Astro Zero-JS & Shadcn/UI)

**Fokus:** Pembangunan situs utama portal masyarakat sesuai Juknis 2026 yang mengedepankan performa ekstrim dan aksesibilitas _blind-friendly_.

- **Langkah 3.1:** Setup ekosistem inti _Astro SSR_ dengan adaptor peladen _Bun_ di `apps/web-portal`.
- **Langkah 3.2:** Integrasi pondasi _Tailwind CSS_ dan perekatan fungsi penyerap Tema Dinamis via _CSS Variables_ dari Database.
- **Langkah 3.3:** Perakitan _Layout Master_: _Header, Footer, Navbar Bertingkat_, dan laci usap navigasi (_Mobile Hamburger Menu_).
- **Langkah 3.4:** Penyambungan tipe data Backend -> Frontend E2E meniru pustaka internal menggunakan ekstensi tRPC-like (_Eden Treaty_).
- **Langkah 3.5:** Desain halaman spesifik Utama (Grid Layanan, Tata letak 2 kolom Berita + Pengumuman Tembok Iframe SIPP).
- **Langkah 3.6:** Penyediaan Pustaka Komponen penampil berkas `<ReportViewer>` reaktif sesuai _Mode Tampil_ (Iframe Dalam / Kartu Pratinjau / Tombol Unduh / Link Eksternal).

---

## Tahap 4: Dasbor Admin Dinamis & Form Builder (React Islands)

**Fokus:** Merancang Pusat Kendali operasional yang membebaskan institusi untuk merubah tata letak dan struktur web tanpa memutus kontinuitas baris kode (Zero Code Edits).

- **Langkah 4.1:** Perancangan antarmuka _Layout Builder_ (Merakit/mengubah urutan blok _Homepage_, pengaturan kuantitas Berita, slider gambar _hero_).
- **Langkah 4.2:** Pembuatan modul _Navigation Menu Tree Builder_ berlapis, mencakup opsi pembukaan Tab Baru dan Otorisasi akses antar tingkatan peradilan.
- **Langkah 4.3:** Mengirim Modul Teks Kaya (_Rich Text Editor_) beserta fungsionalitas pengunggah foto sejalur dengan mitigasi kerentanan XSS (_DOMPurify_).
- **Langkah 4.4:** Modul Manajemen _Sub-Kepaniteraan_ dan _Bagan Hierarki Unit Pengadilan_ yang menyesuaikan rupa secara kondisional (Mode Tinggi vs Negeri vs Tipikor vs PHI).
- **Langkah 4.5:** Pembuatan fitur Pembangun Formulir Dinamis (_Dynamic Form Builder_) untuk Survey IKM/IPAK beserta laporan Excel/PDF khusus _Admin Layanan_.

---

## Tahap 5: Modul Kepaniteraan Spesifik & Sertifikasi Mutu

**Fokus:** Melengkapi persyarat absolut MA dan modul dokumentasi pengawasan internal (_Zona Integritas_ dan _AMPUH_).

- **Langkah 5.1:** Modul pengujian kelarasan URL SIPP, e-Berpadu, JDIH, SP4N-LAPOR, DIREKTORI PUTUSAN sebelum disimpan dalam konfigurasi.
- **Langkah 5.2:** Pusat Dokumentasi Akuntabilitas Tata Kelola: Unggahan DIP (Daftar Informasi Publik) dwi tahunan dan LHKPN KPK.
- **Langkah 5.3:** Pembangunan Ekosistem _Zona Integritas (ZI)_ 6 Area Pemerataan Administratif beserta lencana pengukur progres (Progres Bar Dinamis).
- **Langkah 5.4:** Pembangunan Ekosistem _AMPUH (Penjaminan Mutu Nasional)_ 9 Kriteria lengkap bersama status _Valid_, _Expired_, dan Modul Sertifikasi Mandiri Paripurna.
- **Langkah 5.5:** Sistem penampungan Tiket Layanan Publikasi/Konsultasi Hukum interaktif _(Two-Way Message)_ dilengkapi otentikasi mandiri Publik NIK/OTP WhatsApp masyarakat.

---

## Tahap 6: Ekstraksi Analitik, Optimalisasi Lanjut & Go-Live Skenario

**Fokus:** Memastikan kepatuhan terhadap standar keamanan siber, aksesibilitas difabel 100/100 _Lighthouse Google_, serta mitigasi gagal sistem.

- **Langkah 6.1:** Uji coba resiliensi aksesibilitas (_Accessibility WCAG Auditing_) rasio kontras warna layar serta mode penyemat navigasi tunanetra (_Screen Reader ARIA attributes_).
- **Langkah 6.2:** Penerapan fitur penyuaraan SEO otomatis mumpuni berbasis Tag Open Graph _JSON-LD HTML Semantik_.
- **Langkah 6.3:** Ujicoba sistem Cron Harian Alpine Docker (_Backup PostgreSQL Automation_) lalu melatih pembukaan ekstraksi `Manual Trigger API` lewat _Dashboard Backend_.
- **Langkah 6.4:** Simulasi Latihan Ujung (_Final Deployment Rehearsal_): Verifikasi pencetakan instalasi awal mulus pada sebuah _env_ kosong (Hanya mengandalkan perintah `docker compose up -d`).

---

_Status Dokumen: Disempurnakan dan Siap Dieksekusi._
