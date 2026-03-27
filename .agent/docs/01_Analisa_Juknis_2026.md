# 01. Analisa Lengkap Standarisasi Website Pengadilan Tinggi & Pengadilan Negeri 2026

Berdasarkan Keputusan Direktur Jenderal Badan Peradilan Umum Nomor: **127/DJU/SK.HM1.1/III/2026**, berikut adalah analisis lengkap ketentuan untuk **Pengadilan Tinggi (PT) dan Pengadilan Negeri (PN)**.

## 1. Karakteristik Visual & UI/UX

- **Warna Utama**: Merah Gelap (#9a2109). Dikelola terpusat via Vanilla CSS Variables untuk kemudahan modifikasi admin.
- **Tipografi**: Profesional, bersih, dan konsisten.
- **Layout**: Sederhana, maksimal 2 klik untuk mencapai informasi utama.
- **Responsivitas**: Wajib optimal di Desktop, Tablet, dan Smartphone.
- **Slider**: Maksimal 3-5 slide untuk konten krusial.

## 2. Struktur Menu & Konten 

- **Halaman Beranda**:
  - Tautan Cepat: SIPP, E-Court, e-Berpadu, Direktori Putusan, SISUPER, SIWAS.
  - **Khusus Pengadilan Tinggi (PT)**: Tautan ke seluruh Pengadilan Negeri (PN) di bawah wilayah hukumnya.
  - **Khusus Pengadilan Negeri (PN)**: Tautan ke Pengadilan Tinggi yang membinanya.
  - Indeks Pelayanan: IKM & IPAK (Format SISUPER).
  - **Jadwal Sidang**: Penempatan terintegrasi menggunakan iFrame langsung ke sistem SIPP.
  - **Autentikasi Masyarakat**: Fitur eksklusif Login untuk masyarakat umum agar dapat memantau tiket layanan.
- **Profil**: Visi-Misi, Sejarah, Struktur Organisasi, Wilayah Yurisdiksi, Denah, Profil Hakim/Pegawai. (Ditambah Role Model & Agen Perubahan sesuai Juknis).
  - *Manajemen Kepaniteraan*: Layanan spesifik (Tipikor, Perikanan, PHI, Niaga) disesuaikan dinamis tergantung kelas Pengadilan.
- **Layanan Publik**: PTSP (Jenis, Standar, Maklumat, Kompensasi), Layanan Disabilitas (Wajib Video Juru Bahasa Isyarat).
- **Informasi Publik**: Berkala, Serta-Merta, Tersedia Setiap Saat, Informasi Dikecualikan (Sesuai SK KMA 2-144/2022).
- **Reformasi Birokrasi**: Zona Integritas (ZI) dan Sertifikasi AMPUH.

## 3. Standar Aksesibilitas (WCAG 2.1)

- Kontras warna minimal 4.5:1.
- Dukungan Screen Reader (Alt text pada gambar, struktur Heading H1-H3).
- Navigasi Keyboard-only.
- Pembesaran teks hingga 200% tanpa merusak layout.
> *Capaian WCAG didukung eksekusi HTML semantik sempurna tanpa beban JS membludak berkat Astro.*

## 4. Keamanan & Performa

- **Protokol**: Wajib HTTPS (SSL/TLS).
- **Performa**: Load time < 3 detik (Sangat efisien diproses Bun + ElysiaJS).
- **Backup**: Harian/Mingguan (Terpisah dari server utama). Diakomodir penuh menggunakan cron stack Docker terpusat.
- **Audit**: Vulnerability scanning setiap 6-12 bulan.
