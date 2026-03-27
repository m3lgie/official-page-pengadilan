# 12. Strategi Backup, Pemulihan (Restore) & Manajemen Eksekusi Cadangan

Dokumen ini adalah SOP (Standard Operating Procedure) bagi arsitektur integrasi Cron Backup `pg-backup` yang tertanam dalam file master sinkronisasi `docker-compose.yml`.

## 1. Topologi Layanan Autentik Cron Backup

Mekanisme pencadangan terstruktur untuk platform ini memanfaatkan efisiensi dari **`prodrigestivill/postgres-backup-local`** alpine-container. Spesifikasinya:
- **Jadwal Mutlak**: Setiap Jam **02:00 AM** serentak (*Server Time* berdasarkan sinkronisasi NTP `tzdata`).
- **Retensi (Auto-Prune)**:
  - Menyediakan *7 kopi* harian.
  - Menyediakan *4 kopi* mingguan.
  - Menyediakan *6 kopi* bulanan.
- **Lokasi Fisik Harddisk**: Foldernya di-*mount* ke host pada lokasi `/home/website/backups`.

## 2. Eksekusi Backup Sinkron dari UI Dasbor Admin (Manual Trigger)

Berdasarkan kesepakatan Juknis dan kepraktisan Admin, proses di atas juga dimodifikasi agar dapat ditembak sewaktu-waktu oleh Super Admin (**Manual Execute**):

**A. Tombol React Island pada UI**
Tombol "Amankan Database" menembak RPC ke Elysia: `api-server/backup/trigger`.

**B. Pengaturan Modul `api-server` Backend (Bun Script)**
Karena container `api-server` Backend Elysia memiliki otorisasi internal terhadap Postgres, Elysia akan dibebankan sebuah skrip bawaan Bun:
```typescript
import { $ } from "bun";
import { join } from "path";

// Elysia Endpoint (Murni internal guard auth)
app.post('/api/trigger-backup', async () => {
    const fileName = `manual_dump_${Date.now()}.sql.gz`;
    const outputPath = join('/app/backups', fileName);

    console.log(`Menjalankan manual pg_dump ke ${outputPath}...`);
    // Memanggil binary pg_dump di host/container
    await $`pg_dump -h db -U postgres -d pengadilan_db | gzip > ${outputPath}`;
    
    return { success: true, file: fileName };
});
```
File *dump* akan muncul di antarmuka Dasbor Admin untuk diunduh.

## 3. Playbook Pemulihan Bencana (Catastrophe Restoration SOP)

Secara teori, data sangat jarang membusuk pada sistem PostgreSQL terkini, namun jika institusi terkena imbas serangan ransomware berskala sistem pada mesin host:

Lakukan pemulihan selektif dari berkas backup:

1. **Bersihkan Database yang Menurun Kualitasnya**:
   ```bash
   docker-compose exec -u postgres db psql -c "DROP DATABASE pengadilan_db;"
   docker-compose exec -u postgres db psql -c "CREATE DATABASE pengadilan_db;"
   ```

2. **Injeksikan file `.sql.gz` ke dalam Jantung PostgreSQL**:
   Contoh format skrip unzipper murni:
   ```bash
   gunzip -c backups/daily/2026-03-27.sql.gz | docker-compose exec -T db psql -U postgres -d pengadilan_db
   ```
   *Peringatan*: Pastikan anda menjalankannya HANYA menggunakan parameter `-T` (Non-TTY alias pipe bypass) dan Drizzle ORM serta Astro wajib dimatikan (`docker-compose stop api-server web-portal`) kala proses sedot berlangsung.
