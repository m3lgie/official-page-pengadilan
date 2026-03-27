# 06. Admin Panel & Management System Specification

Admin Panel adalah pusat kendali semua konten dan tata letak dinamis Pengadilan. Pada The Speed Stack, Dashboard Admin ini berjalan sebagai *React Island* interaktif di atas Astro, ber-API ke Elysia.

## 1. Manajemen User (Identity & Access)

- **Mode Multi-Tenant Global**: Tombol *toggle* terpusat (Global Setting) untuk membongkar pasang mode *Pengadilan Tinggi* ke *Pengadilan Negeri*, lengkap dengan limitasi fitur Kepaniteraan opsional.
- **Akun Publik Khusus**: Sub-menu manajemen akun pengguna publik (masyarakat umum) agar Admin bisa mereset password warga yang berurusan di meja PTSP apabila mereka tidak dapat *login*.

- **Role-Based Access Control (RBAC)**:
  - **Super Admin**: Kendali penuh sistem, manajemen user, eksekusi System Backup SQL-Gzip Manual, dan log audit.
  - **Editor/PPID**: Mengelola konten berita, laporan, dan informasi publik (DIP).
  - **Admin Layanan**: Mengelola jadwal sidang (Iframe Config) dan data layanan PTSP.
- **Keamanan (Backend Bun Middleware)**:
  - **Individual Accounts**: Dilarang menggunakan akun bersama.
  - **2FA (Two-Factor Authentication)**: Wajib diaktifkan untuk semua level admin.
  - **Password Policy**: Enkripsi Argon2 (Masa berlaku 3-6 bulan).

## 2. Manajemen Konten (CMS Modules)

- **Module Berita & Publikasi**:
  - Editor Rich Text (WYSIWYG) dengan dukungan alt-text untuk gambar.
  - Workflow persetujuan (draft -> review -> publish) terekam Drizzle ORM.
  - Kategorisasi (Berita, Artikel, Pengumuman).
- **Module Profil Satker**:
  - Manajemen data Hakim & Pegawai (Foto, Jabatan, Riwayat Pendidikan & Karier).
  - Update Visi, Misi, Sejarah, Struktur Organisasi, Profil Role Model & Agen Perubahan.
- **Module Laporan Publik**:
  - Upload PDF terintegrasi (DIPA, LHKPN, LKjIP) dengan validasi tipe berkas aman anti-executable shell.
  - Manajemen Daftar Informasi Publik (DIP) per 6 bulan rutin.

## 3. Module Khusus Pengadilan Tinggi & Negeri (Strategic)

- **Jurisdiction Manager (PT Base)**: Daftar tautan dan koordinasi website PN di bawah wilayah hukum PT.
- **Service Widget Integration**:
  - Konfigurasi URL vital SIPP, E-Court, e-Berpadu, dan SIWAS.
  - Kontrol otomatis Iframe "Jadwal Sidang SIPP" dari luar untuk ditempel di HomePage.
  - Manajemen Jam Layanan PTSP dan Maklumat Pelayanan.
- **Disability Support Module**:
  - Pengaturan tautan khusus video Juru Bahasa Isyarat untuk setiap layanan utama PTSP.
  - Toggle mode aksesibilitas tinggi (High-Constrast/Font Enlargement).

## 4. Audit & Monitoring (Security Compliance)

- **Audit Trail Log**: Pencatatan setiap aksi admin (siapa, kapan, apa yang diubah) disimpan presisi milidetik oleh `Bun.file()` minimal retensi 1 tahun.
- **Status Monitoring Dashboard**: Menampilkan *uptime* website dan ketersediaan (ping status) tautan layanan utama eksternal (SIPP, e-Court, dll).
