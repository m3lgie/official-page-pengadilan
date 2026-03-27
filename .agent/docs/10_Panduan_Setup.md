# 10. Panduan Setup & Instalasi (Multi-Satuan Kerja)

Panduan ini memandu siapapun (admin IT satuan kerja) untuk menginstal sistem dari nol di server pengadilan manapun — cukup dengan Docker.

## Prasyarat Server

| Kebutuhan | Minimum | Rekomendasi |
|---|---|---|
| OS | Ubuntu 20.04 LTS | Ubuntu 22.04 LTS |
| RAM | 2 GB | 4 GB |
| Storage | 20 GB | 50 GB SSD |
| Docker | v24+ | v26+ |
| Docker Compose | v2.20+ | v2.27+ |
| Port Terbuka | 80, 443 | 80, 443, 22 (SSH) |

---

## Langkah Instalasi (Hanya 3 Perintah Utama)

### Step 1 — Salin & Konfigurasi Environment

```bash
# Salin template konfigurasi
cp .env.example .env

# Edit sesuai data pengadilan Anda
nano .env
```

**Wajib diisi di `.env`:**
```bash
SITE_URL=https://pn-namakota.go.id          # Domain resmi pengadilan
DB_PASSWORD=password_kuat_minimal_20_karakter
REDIS_PASSWORD=password_redis_kuat
JWT_SECRET=$(openssl rand -hex 32)           # Generate otomatis
PUBLIC_JWT_SECRET=$(openssl rand -hex 32)    # Generate otomatis
```

### Step 2 — Jalankan Seluruh Stack

```bash
docker compose up -d --build
```

Perintah ini akan otomatis:
- ✅ Build image `api-server` (Bun + Elysia)
- ✅ Build image `web-portal` (Astro SSR)
- ✅ Start PostgreSQL 15 + Redis 7
- ✅ Start Nginx sebagai reverse proxy
- ✅ Mengaktifkan pg-backup (jadwal harian 02:00)

### Step 3 — Inisialisasi Database

```bash
# Masuk ke container API Server
docker exec -it pengadilan_api sh

# Jalankan migrasi Drizzle (membuat semua tabel)
bun run db:push

# Jalankan seeder data awal (dikonfirmasi 'y')
bun run db:seed

# Keluar dari container
exit
```

Setelah `db:seed` selesai, sistem siap dengan:
- Akun Super Admin default (ubah password segera!)
- Data konfigurasi awal `global_configs` (Nama PT, logo placeholder)
- 9 Kriteria AMPUH + 6 Area ZI sudah terisi otomatis

---

## Struktur File Setelah Instalasi

```text
/home/website/
├── docker-compose.yml        ← Orkestrasi utama (tidak perlu diedit)
├── .env                      ← RAHASIA — Konfigurasi satuan kerja ini
├── .env.example              ← Template referensi
├── infra/
│   ├── nginx/
│   │   ├── nginx.conf        ← Konfigurasi global Nginx
│   │   ├── conf.d/
│   │   │   └── pengadilan.conf  ← Virtual host (edit domain di sini)
│   │   └── ssl/              ← Letakkan file .crt dan .key di sini
│   └── docker/
│       ├── api-server.Dockerfile
│       └── web-portal.Dockerfile
└── apps/
    ├── api-server/           ← Source code Backend
    └── web-portal/           ← Source code Frontend
```

---

## Konfigurasi SSL/HTTPS (Opsional tapi Sangat Disarankan)

```bash
# Install Certbot (Let's Encrypt gratis)
apt install certbot

# Generate sertifikat
certbot certonly --standalone -d pn-namakota.go.id

# Salin ke folder SSL Nginx
cp /etc/letsencrypt/live/pn-namakota.go.id/fullchain.pem ./infra/nginx/ssl/
cp /etc/letsencrypt/live/pn-namakota.go.id/privkey.pem ./infra/nginx/ssl/
```

Kemudian aktifkan blok SSL di `infra/nginx/conf.d/pengadilan.conf` (hapus komentar `#`) lalu restart:

```bash
docker compose restart nginx
```

---

## Konfigurasi Identitas Satuan Kerja (dari Admin Panel)

Setelah instalasi selesai, Admin Super buka `https://domain-anda.go.id/admin/pengaturan` dan konfigurasikan:

| Pengaturan | Keterangan |
|---|---|
| Mode Tenant | Toggle: Pengadilan Tinggi / Pengadilan Negeri |
| Nama Pengadilan | e.g. "Pengadilan Negeri Surabaya" |
| Kelas Pengadilan | IA Khusus / IA / IB / II |
| Logo | Upload logo resmi (PNG/WebP) |
| Warna Primer | Hex code warna khas pengadilan |
| Badge WBK/WBBM | Aktifkan/nonaktifkan badge |
| URL SIPP | Tautan iFrame Jadwal Sidang |
| URL Layanan | e-Court, e-Berpadu, SIWAS, dll |

**Tidak perlu menyentuh kode sama sekali!**

---

## Perintah Operasional Berguna

```bash
# Lihat status semua container
docker compose ps

# Lihat log realtime
docker compose logs -f

# Lihat log spesifik service
docker compose logs -f api-server
docker compose logs -f web-portal

# Restart satu service (misal setelah update config)
docker compose restart nginx

# Update aplikasi ke versi terbaru (pull repo → rebuild)
git pull
docker compose up -d --build

# Trigger backup manual
docker exec pengadilan_api bun run backup

# Masuk ke shell PostgreSQL
docker exec -it pengadilan_db psql -U postgres -d pengadilan_db

# Cek file backup yang tersimpan
ls -lh $(docker volume inspect pengadilan_backups | grep Mountpoint | awk -F'"' '{print $4}')
```

---

## Rollback & Pemulihan Data

```bash
# Restore dari file backup .sql.gz
docker exec pengadilan_api sh -c \
  "gunzip -c /app/backups/NAMA_FILE.sql.gz | psql $DATABASE_URL"
```

---

## Topologi Jaringan Docker

```
Internet
    │
    ▼
[Nginx :80/:443]  ← Satu-satunya titik masuk publik
    │
    ├──► [web-portal :4000]  ← Astro SSR (internal only)
    │         │
    │         └──► [api-server :9090]  ← Bun + Elysia (internal only)
    │                   │
    │              ┌────┴────┐
    │          [db :5432]  [redis :6379]  (internal only)
    │
    └── [pg-backup]  ← Akses DB langsung untuk pg_dump
```

Semua container berada di network `internal` (bridge). Hanya Nginx yang menerima koneksi dari luar.
