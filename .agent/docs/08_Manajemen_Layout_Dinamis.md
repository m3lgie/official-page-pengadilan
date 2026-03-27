# 08. Dynamic Layout Management Specification

Sistem ini memungkinkan Administrator (Super Admin/Editor) untuk mengatur tata letak halaman utama secara dinamis tanpa mengubah kode situs, difasilitasi keandalan **Astro SSR**.

## 1. Konsep "Block-Based Layout"

Halaman beranda dibagi menjadi beberapa blok yang dapat diaktifkan, dinonaktifkan, dan diatur urutannya (Drag & Drop).

### Blok Utama:

- **Hero Slider**: Pengaturan gambar slide, teks overlay, dan link tujuan.
- **Quick Services Grid**: Manajemen 6 icon layanan utama (E-Court, SIPP, dll).
- **Main News Grid**: Pengaturan jumlah berita yang tampil (3, 6, atau 9) dan kategori sumbernya ditarik dari Redis/Elysia.
- **Announcement List**: Daftar pengumuman terbaru dengan filter kategori.
- **Jadwal Sidang Widget**: Pengaturan URL Iframe SIPP atau integrasi jadwal eksternal berbasis *string config*.
- **RSS Feed Aggregator**: Input URL RSS dari Mahkamah Agung dan Direktorat Jenderal.

## 2. Sidebar Widget Manager

Admin dapat mengatur kemunculan widget di sisi kanan halaman (Disesuaikan juga berdasarkan Mode Yurisdiksi PT/PN):

- **Search Bar**: Toggle ON/OFF.
- **Language Switcher**: Pengaturan bahasa (ID/EN).
- **Working Hours**: Input jam layanan operasional PTSP secara presisi.
- **Service Index Widget**: Input manual angka kepuasan atau sinkronisasi data otomatis IKM/IPAK (Format SISUPER).
- **Social Media Icons**: Manajemen link ke Facebook, Instagram, Twitter, YouTube.

## 3. Header & Footer Customizer (Berbasis Parameter Sinyal Bun)

- **Branding**: Upload Logo Pengadilan, input Nama Pengadilan, dan pengaturan badge status akreditasi ZI (WBK/WBBM).
- **Contact Info**: Update Alamat, No Telp, Fax, dan Email yang muncul kokoh di Footer statis.

## 4. Teknis Implementasi (Astro SSR & React)

- **Layout Component**: Komponen Layout (`.astro`) akan melakukan pengambilan JSON *layout state* dari respons API Elysia untuk menentukan penempatan baris komponen.
- **Astro Islands (Hydration)**: Menggunakan Island Architecture untuk meminimalkan beban JavaScript di *client-side*. *Language Switcher* atau *Search Bar* yang rumit direkayasa menggunakan React, sisanya berdiri *Zero-JS* HTML murni.
- Pengaturan publik/privat ini bereaksi secara ajaib apabila terdapat interaksi dari pengguna sipil *(Public Authentication)* mendeteksi tiket mereka terhubung di Header.
