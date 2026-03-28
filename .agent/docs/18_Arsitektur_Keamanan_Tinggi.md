# 18. Arsitektur Keamanan Tingkat Tinggi (Enterprise Security)

Sistem Informasi Pengadilan wajib memenuhi spektrum keamanan tingkat lanjut untuk mencegah kebocoran data (Data Breach), manipulasi konten publik (Defacement), maupun kelumpuhan masif (DDoS).

Dokumen ini melengkapi `02_Standar_Portal_Pemerintah.md` dengan penetapan standar pengamanan teknis (Security Architecture) di setiap lapisan aplikasi (The Speed Stack).

---

## 1. Lapisan Infrastruktur & Jaringan (Perimeter Security)

- **Isolasi Database (Network Bridging)**: PostgreSQL (`db`) dan Redis (`redis`) **DILARANG** membuka port ke akses publik internet (misal: 5432 atau 6379 tidak boleh diekspos di `docker-compose.yml` `ports:`). Keduanya murni jalan di dalam Docker `internal` network.
- **Nginx sebagai Perisai Utama**: Nginx bertugas ganda sebagai _Reverse Proxy_ dan fasilitator WAF (_Web Application Firewall_).
- **Pembatasan Rate Limiting Mutlak**: Nginx tidak boleh membiarkan _brute force_. Tetapkan limit:
  - Frontend Astro (HTML/Aset): 100 req/s per IP.
  - API Bun Elysia (`/api`): 30 req/s per IP.
  - Endpoint Autentikasi (`/api/auth/*`): 5 req/min per IP (Melumpuhkan _password guessing/credential stuffing_).

## 2. Lapisan Basis Data & Kriptografi (Data at Rest)

- **One-Way Password Hashing**: HANYA **Argon2 / Argon2id** yang sah digunakan dalam parameter pendaftaran dan masuk sistem. Sandi _MD5, SHA1_, atau bahkan _Bcrypt_ tidak diizinkan.
- **Isolasi Kunci JWT (JSON Web Token)**: JWT untuk masyarakat (warga pencari keadilan) WAJIB ditandatangani menggunakan `PUBLIC_JWT_SECRET` yang berbeda kuncinya (_key_) dengan `JWT_SECRET` milik Pejabat/Admin internal. Akses tak beruntun akan ditolak saat dekode berlapis `authGuard`.
- **Masa Berakhir Token (TTL)**:
  - Admin Token: 8 Jam (Maksimum durasi shift pegawai purna waktu).
  - Public Token: 2 Jam + Mekanisme _Refresh Token_ jika dibutuhkan.

## 3. Eksekusi Lapisan Aplikasi & Input (Elysia Validation)

- **End-to-End Type Safety dengan TypeBox**: Elysia menolak 100% _payload_ JSON masukan yang disuntikkan skrip _(XSS)_. Aturan utama: Gunakan `t.Object({ ... }, { additionalProperties: false })` pada setiap rute validasi agar _attacker_ gagal menyisipkan kolom bodong yang menyebabkan eskalasi previlese (seperti `{"role": "SUPERADMIN"}`).
- **Sanitasi HTML Tingkat Mahkamah**:
  - Admin dapat mengisi artikel dengan `RichTextEditor`.
  - Bun _Server_ WAJIB menggunakan modul sanitasi XSS (e.g. `isomorphic-dompurify` atau `sanitize-html`) untuk membilas _(purge)_ _tag_ `<script>`, `onload`, `onerror` sebelum data dibuang ke Drizzle ORM.
- **Blokir SSRF (Server-Side Request Forgery)** pada Iframe:
  Sistem memperbolehkan simpan URL untuk iframe Jadwal Sidang (SIPP). URL yang diunggah harus melewati _Regex_ yang memastikan domain yang didukung (misal `.go.id` atau `.mahkamahagung.go.id`). Situs asusila atau periklanan otomotis ditolak tipe filternya.

## 4. Keamanan Unggahan Berkas (Secure File Uploads)

Modul `media_assets` rentan diinfeksi skrip _Web Shell_. Mitigasi teknis terberatnya:

1. **Renaming**: Nama file asli tidak pernah disimpan secara utuh atau disajikan secara langsung. Semuanya disandikan menyadur UUID murni `v4()`. (Contoh: `laporan_keuangan.php` -> `550e8400-e29b-41d4-a716-446655440000.webp`).
2. **Strict MIME-Type Checking**: Membaca _header magic bytes_, bukan sekadar ekstensi di ujung nama file (ekstensi `.pdf` padahal kontennya `.sh` akan otomatis tertolak). Elysia menggunakan TypeBox File validation (`t.File()`).
3. **No-Exec Location**: Volume Docker `pengadilan_uploads` murni dioperasikan (_mounted_) pada Nginx block _location_ untuk dirender sebagai statis tak terkomputasi. Izin sistem ditiadakan.

## 5. Keamanan Aksesibilitas Admin & Audit

- **2FA (Two-Factor Authentication)**: Fitur OTP disokong WhatsApp aktif secara wajib (hukum positif) untuk _Super Admin_ ketika memengaruhi/bermain dengan _Data DIPA / Laporan Tahunan_.
- **Trail Forensik (Audit Logs)**: Tabel `audit_logs` akan merekam remah jejak Admin sebelum perubahan data (`payload_before`) dan setelah perubahan data (`payload_after`). Log ini bersifat _append-only_ (hanya sisip, hapus dilarang keras, bahkan oleh relasi ORM). Keamanan non-repudiasi.
- **Cross-Origin Resource Sharing (CORS)**: Dilarang memakai (_Origin:_ `*`). Secara tegas hanya `SITE_URL` alias parameter `env` domain institusi yang dapat melakukan pemanggilan `POST/PATCH/DELETE` ke API Bun.

Sistem Anda kini tak sekadar "mencetak layar HTML SSR", melainkan beroperasi bak dinding besi arsitek perbankan demi nama baik Mahkamah Agung.
