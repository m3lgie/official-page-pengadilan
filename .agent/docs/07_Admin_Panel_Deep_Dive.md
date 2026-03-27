# 07. Deep Dive: Dynamic Admin Panel Specification

Filosofi utama: **"Code Once, Configure Anywhere"**. Seluruh aspek operasional dan visual dapat diatur tanpa intervensi developer, dijalankan dengan arsitektur canggih *React Islands* dan pemanggilan E2E Elysia *Eden Connector*.

## 1. Dynamic Navigation & Menu Builder

- **Tree Structure**: Drag-and-drop builder (*Island React*) untuk menyusun Menu Utama, Sub-Menu, hingga level ke-3.
- **Validasi Mutlak Juknis**: Memastikan *Parent Menu* esensial Peradilan tidak bisa dihapus, hanya bisa dimodifikasi.
- **Link Types**: Pilihan link internal (halaman/berita), link eksternal (SIPP, e-Court), atau anchor link.
- **Visibility Toggle**: Menyembunyikan/menampilkan menu berdasarkan status (aktif/non-aktif).

## 2. Dynamic Page Builder (Static Content)

- **Custom Pages**: Kemampuan membuat halaman statis baru (misal: "Halaman Inovasi Khusus" atau "Profil Pimpinan Baru").
- **Template Blocks**: Memilih susunan blok (Text, Image, Accordion untuk FAQ, Video) per halaman (Rendered by Astro).

## 3. Global Configuration Module

- **Platform Integrasi**: Satu tempat untuk mengupdate *URL Iframe Jadwal Sidang*, e-Berpadu, SIWAS, dan API key lainnya.
- **Sistem Ganda Pengadilan**: Centang cepat mengatur UI keseluruhan antara corak "Pengadilan Tinggi" atau "Pengadilan Negeri".
- **SEO & Analytics Manager**:
  - Input Script Injection (Header/Footer) untuk tracking code tanpa menyentuh *source files*.
  - Manajemen Robot.txt dan XML Sitemap generation mandiri.
- **Maintenance Mode**: Sakelar (Toggle) absolut untuk mengaktifkan halaman *offline/pemeliharaan* dengan pesan izin kustom dari super admin.

## 4. Advanced User & Role Management

- **Granular Permissions**: Pengaturan izin sangat spesifik per modul (misal: "Admin A bisa buat berita tapi tidak bisa publish", "Admin B hanya bisa ubah Iframe SIPP").
- **Audit Trail Visualization**: Dashboard khusus (Recharts statis) untuk melihat log aktivitas admin secara visual (filter berdasarkan user, modul, rentang waktu).

## 5. Dynamic Form Builder & Interaksi Masyarakat

- **Custom Inputs**: Kemampuan CMS membuat form seperti *Kontak, Pengaduan Internal Terpadu, atau Survey SPBE* dengan pilihan input *Teks, Dropdown, Checkbox, dan Lampiran Unggahan PDF/Image* melalui *Drag & Drop antarmuka*.
- **Public Integration Account**: Form yang dipublikasi dapat dijawab langsung oleh Warga Publik yang telah *Login* demi keamanan *submission*.
- **Result Management**: Dashboard interaktif memeriksa hasil tanggapan masyarakat, sekaligus mengekspor rekapitulasinya ke format **Excel/PDF**.

## 6. Theme & Branding Engine (Full Color Control)

Sistem pengaturan visual yang mendalam (*Astro CSS Variables*) untuk menjaga konsistensi identitas lembaga:
- **Global Theme Settings**: Pengaturan warna utama (*Primary* `misal #9a2109`), sekunder, dan aksen yang mempengaruhi seluruh situs.
- **Header & Footer Customizer**: Pengaturan warna latar belakang, warna teks, transparansi.
- **Module-Specific Styling**: Kemampuan memberikan identitas aksen warna yang berbeda per modul.
- **Visual Assets**: Manajemen Logo (Normal, Monochrome, Inverted) dan Favicon.
