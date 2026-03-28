# 17. Analisis Kendala & Mitigasi (The Speed Stack)

Dokumen ini berisi analisis tingkat kesulitan dan mitigasi potensial kendala (bottlenecks) dalam pembangunan Sistem Website Pengadilan 2026. Dokumen ini sejalan dengan _Knowledge Sync_ dan wajib dibaca sebelum memulai tahapan integrasi fitur-fitur dinamis.

## 1. Tingkat Kesulitan: TINGGI (8.5 / 10)

Spesifikasi sistem ini mengarah pada arsitektur _Enterprise-Grade_ dengan tingkat kedinamisan yang sangat tinggi (Prinsip _Zero-Hardcode_).

- **Arsitektur Multi-Tenant Dinamis**: Sistem berubah bentuk berdasarkan nilai di `global_configs` (`tenant_mode` PT/PN, `court_class`). Logika _routing_ dan _rendering_ komponen harus dibuat reaktif secara kondisional tanpa ada kebocoran menu antar konfigurasi.
- **Skala Database (14 Domain, 50+ Tabel)**: Drizzle ORM menjamin _type-safety_, namun relasi 50 tabel (termasuk CMS Berita, ZI, AMPUH, Layanan, Form Builder) membutuhkan pengoptimalan kueri yang sangat matang agar _Time to First Byte_ (TTFB) tetap kencang.
- **State Management di ekosistem "React Islands"**: Astro memisahkan HTML statis dan JS interaktif. Membangun fitur besar seperti _DragDropLayoutEditor_ atau _FormBuilder_ tanpa membebani performa _First Load_ menuntut strategi _hydration_ (`client:load`, `client:idle`, atau `client:visible`) yang sangat selektif.

## 2. Titik Kritis (Bottlenecks) & Strategi Mitigasi

### A. Manajemen Cache Redis vs SSR (Astro + Elysia)

- **Kondisi**: Astro berjalan murni dalam mode SSR (_Server-Side Rendering_) karena identitas pengadilan (logo, teks, layout) ditarik dinamis dari PostgreSQL. Jika tiap muat-ulang halaman menembak DB, metrik LCP (Lighthouse) akan gagal total.
- **Mitigasi**: Implementasi arsitektur **Precise Cache Invalidation** di Elysia. Endpoint `/api/global/config` di-_cache_ di Redis (TTL 5 Menit). Setiap kali _Superadmin_ melakukan `PATCH` ke konfigurasi/layout, sebuah _hook_ wajib membuang kunci Redis tersebut tepat pada detik itu juga (_purge_).

### B. Ujung ke Ujung Type-Safety (Drizzle -> Elysia/TypeBox -> Eden Treaty)

- **Kondisi**: Proyek ini mewajibkan TypeScript _Strict Mode_ total tanpa ada tipe `any` atau `ts-ignore`.
- **Mitigasi**: Pada fitur hiper-dinamis seperti _Form Builder_ (di mana _payload submit form_ JSON dari pengguna tidak pasti kerangkanya), mendefinisikan skema dinamis di TypeBox akan sangat menyengsarakan. Solusinya, buat struktur validasi JSON yang melonggarkan cek anak-simpul (hanya struktur _parent_ form-nya) pada skema pertama, kemudian divalidasi manual lapis kedua mengikuti definisi skema form di database.

### C. Konflik CSS Variables (Theme Engine) dengan Tailwind & Shadcn/UI

- **Kondisi**: Admin dapat mengganti `primary_color` (format HEX) dari antarmuka Web. Warna ini otomatis diaplikasikan ke seluruh antarmuka komponen.
- **Mitigasi**: Komponen Shadcn memakai standar warna format HSL (`--primary: 222.2 47.4% 11.2%`). Memasukkan nilai HEX secara mentah akan merusak _opacity modifiers_ (`bg-primary/50`). Wajib meletakkan fungsi utilitas konverter HEX ke HSL di Astro global layout sebelum menulis ke `<style>` lokal variabel CSS.

### D. Kepatuhan Aksesibilitas Mutlak (WCAG 2.1)

- **Kondisi**: Standar mengharuskan nilai Skor Lighthouse _Accessibility_ bernilai **100**.
- **Mitigasi**: Pastikan semua gerak interaktif Shadcn (`Dialog`, `Sheet`/Drawer, `Slider`, `DropdownMenu`) dikunci dengan label `aria-label` yang spesifik di setiap modifikasi. Amankan fitur _Focus Trap_ (fokus _keyboard_ yang terkunci di dalam modal saat muncul) pada mode layar _Mobile_.

### E. Keamanan Eksekusi Konten Dinamis (XSS Vulnerability)

- **Kondisi**: Adanya elemen `<RichTextEditor>` untuk pembuatan CMS, _script injection_, dan integrasi _Iframe_ e-Court.
- **Mitigasi**: Jangan sekalipun melakukan fungsi reaktif dasar `dangerouslySetInnerHTML` di Astro/React tanpa memastikan struktur konten telah melalui filter HTML (seperti paket DOMPurify/isomorphic) dari Bun Server. Tiap URL _iframe_ yang hendak disimpan di DB harus lolos regex validasi `uriValidator` bahwa ia merujuk ke peladen resmi Mahkamah Agung agar lolos dari jerat _CSRF/Clickjacking_.
