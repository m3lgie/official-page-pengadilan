# 11. Implementation Roadmap - Website Pengadilan (The Speed Stack)

Rencana tahap demi tahap (Milestones) memindahkan rencana dokumentasi "Otak Agen" ke pengerjaan kode dengan *Decoupled Monorepo Workflow*.

## Proposed Implementation Phasing

### Tahap 1: Inisialisasi Arsitektur Dasar (Monorepo Setup)

- **[NEW]** Setup *Bun Workspace* di Folder Root (`package.json`).
- **[NEW]** Mendaulat folder `apps/api-server`. Memasang framework **ElysiaJS**, engine *Bun Core*, dan konektor DB gubahan **Drizzle ORM** (Lengkap schema tabel `global_configs`, `users`, `news`, dsb).
- **[NEW]** Setup folder ekosistem Astro (`apps/web-portal`) termasuk Tailwind CSS dan penyelarasan standar **Shadcn/UI** (`components/ui`).
- **[NEW]** Menyediakan folder lintasan tipe tersentral (`packages/shared-types`). Backend mendelegasi skema type Eden Connector.

### Tahap 2: Komponen Dasar & Layouting (Juknis Sinyal)

- Pembangunan kerangka tata letak (*Layout* global `14_Standar_Layout_Template.md`) dengan warna dinamis hasil kueri dari `global_configs`.
- Menyusun Header, Slider, Grid Berita, RSS, dan Sidebar Utilities dengan Astro Zero-JS + *React*.
- Menyiapkan mode "Toggle Kelas Pengadilan PT/PN" agar frontend beradaptasi terhadap tautan yurisdiksi.

### Tahap 3: Modul CMS Admin Shadcn & Masyarakat Publik

- Integrasi Autentikasi Ganda: Argon2 + Elysia JWT Middleware untuk Admin (Subdomain/Rute Privat) dan Otorisasi Tiket Publik (Masyarakat).
- Menyusun Drag & Drop Menu Builder dan Editor Berita menggunakan kemewahan elemen-elemen dari *Shadcn/UI* di dasbor peradilan.

### Tahap 4: Integrasi Modul Eksternal & Serah Terima

- Mengonversi alamat Iframe SIPP (Jadwal Sidang) berbasis konfigurasi admin tanpa membedah API.
- Fitur Ekspor *Survey IKM SISUPER* ke PDF/Excel.
- Uji Coba Pencadangan *Cron Docker Backup* dan Skema Pertahanan Serangan *DDoS* (WAF Native Caching).

## Verification Checklist

- **Performa Mutlak**: Skor Lighthouse menembus ambang 100/100, FCP < 0.8s untuk Web Portal karena Astro statis render UI yang berat.
- **Aksesibilitas (A11y)**: Test `Axe-core` aman. (Menu disabilitas Bahasa Isyarat berfungsi).
- **Interoperabilitas Codebase**: Terbukti sistemnya sanggup jalan sebagai Pengadilan Tinggi **maupun** Pengadilan Negeri murni berbasis Data Table CMS (Bukan Bedah File).
