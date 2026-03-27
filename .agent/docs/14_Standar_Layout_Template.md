# 14. Standar Layout Template (Juknis 2026 Wireframe)

Berkas ini mengonversi rancangan gambar struktur tata letak (wireframe) Juknis 2026 (`img137.jpg` & `img141.jpg`) menjadi cetak biru implementasi komponen UI untuk Frontend Astro dan Tailwind/Shadcn UI. Desain ini berlaku secara menyeluruh bagi The Speed Stack (Pengadilan Tinggi & Pengadilan Negeri).

## 1. Top Header Area (Area Kepala Utama)

Bagian teratas dari website memuat identitas absolut institusi peradilan.
- **Top-Left (Kiri Atas)**: Logo Pengadilan berbentuk lingkaran. Dikelola dinamis dari konfigurasi CMS.
- **Top-Center (Tengah Atas)**:
  - Teks Baris Pertama: **"MAHKAMAH AGUNG REPUBLIK INDONESIA"** (Teks besar tebal).
  - Teks Baris Kedua: **"PENGADILAN ..."** (Dinamis: Diambil dari entitas `court_name` di `global_configs`).
- **Top-Right (Kanan Atas)**: Dua *badge* lingkaran berdampingan (WBK & WBBM) untuk melambangkan akreditasi instansi.

## 2. Main Navigation Bar (Bilah Navigasi Utama)

Ditampilkan berupa blok *menubox* solid dengan deretan opsi wajib dari kiri ke kanan (ditentukan di `navigation_menus` Drizzle):
- BERANDA
- TENTANG PENGADILAN
- LAYANAN PUBLIK
- LAYANAN HUKUM
- BERITA
- HUBUNGI KAMI
- REFORMASI BIROKRASI

## 3. Hero Section & Quick Links (Komponen Visual Pembuka)

Segera setelah menu utama, terdapat dua lapisan krusial:
1. **Slide Banner**: Kotak *carousel/slider* lebar penuh yang dirender *client-side hydration* (ukuran proposional) untuk mengedukasi masyarakat terkait *campaign* saat ini.
2. **Grid Tautan Cepat (10 Kotak Grid)**: Deretan *box* (2 x 5) yang tersusun secara merata mencakup Layanan Publik Utama MA:
   - Baris 1: eCourt, eBERPADU, eRATERANG, SIPP, Direktori Putusan.
   - Baris 2: JDIH, LPSE, SIWAS, Inovasi Satker, SISUPER (beserta logo/icon masing-masing).

## 4. Main Body: Mode Kolom Ganda (2-Column Grid)

Beranda depan (Khususnya Layout Beranda Komprehensif `img137`) menerapkan pemisahan **Layout Kiri (Konten Utama) sebesar ~65-70%** dan **Layout Kanan (Sidebar) sebesar ~30-35%**.

### A. Kolom Kiri (Main Content Stream)

Terdiri dari tumpukan blok fungsional yang menggiring mata pengunjung ke bawah:
1. **Maklumat Layanan**: Dua kotak *side-by-side* (*Maklumat Pelayanan* di kiri, *Maklumat Layanan Informasi Publik* di kanan).
2. **Berita Terkini**: Layout *Grid/Card* memuat *Foto Berita, Judul Berita, Tanggal*, dan cuplikan *Berita Singkat*.
3. **Pengumuman**: Blok teks daftar tanpa gambar. Tampilan baris-baris (Tanggal - Judul Pengumuman).
4. **RSS Feeds (Grid Sinkronisasi Lintas Satker)**: Kotak tabular yang menanamkan berita sentral:
   - RSS Berita MA / Ditjen Badilum.
   - RSS Pengumuman MA / Ditjen Badilum.
   - RSS Pengadilan Tinggi (PN wajib mengekspos berita spesifik PT pembinanya).
5. **Jadwal Sidang**: Blok lebar penuh. *Untuk PN (Iframe utuh aplikasi SIPP)*. *Untuk PT (Sesuai output inovasi jadwal PT)*.
6. **Video/Tautan Banner Pengadilan**: Konten pembeda dinamis hasil unggahan satker mandiri (`custom_modules`).
7. **Hubungi Kami (Bottom-Left)**: Disematkan teks sapaan *"Silakan hubungi ..., kami melayani para pencari keadilan dengan sepenuh hati"* berikut detail alamat, telepon, faks, dan email.

### B. Kolom Kanan (Sidebar Tetap / Utility)

Sidebar memiliki elemen utilitas dan metrik informasi statis yang esensial:
1. **Alih Bahasa (Language Switcher)**: Modul React *interactive*.
2. **Pencarian (Search Box)**: Bar masukan teks.
3. **Jam Kerja Pengadilan & PTSP**: Tampilan jam jadwal.
4. **Indeks Pelayanan Publik (IKM/IPAK)**: Render dinamis nilai agregat survei.
5. **Tautan Website Sentral**: Tautan ke *Mahkamah Agung*, *Ditjen Badilum*, dll.
6. **Sosial Media Pengadilan**: Daftar berformat tabel (*Logo* | *Nama Akun*).
7. **Statistik Situs**: Menampilkan jumlah pengunjung total/harian.

## 5. Footer (Bagian Kaki Halaman)

Memiliki garis pembatas yang jelas dengan informasi atribusi tunggal yang merentang seluruh lebar halaman:
- Teks legal standar: **"Copyright ©2026 Pengadilan Tinggi/ Negeri....."** (Nama dinamis satker).

---
*Catatan Bagi Agen/Developer: Template layout murni dari PDF/gambar Juknis ini harus dibangun menggunakan tata letak Tailwind CSS Grid (khususnya untuk rasio sidebar dan daftar quick-links).*
