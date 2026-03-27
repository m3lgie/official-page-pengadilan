# 16. Manajemen Struktur Organisasi Pengadilan

Berdasarkan riset struktur resmi Mahkamah Agung RI, dokumen ini mendefinisikan spesifikasi lengkap fitur manajemen **Struktur Organisasi** untuk PT (Pengadilan Tinggi) dan PN (Pengadilan Negeri) — mencakup database, Admin Panel, dan tampilan Frontend.

---

> ⚠️ **ATURAN DINAMIS WAJIB**: Struktur organisasi PT dan PN **BERBEDA** secara signifikan.
> Sistem DILARANG hardcode jabatan. Semua jabatan dibaca dari database berdasarkan `tenant_mode` dan `court_class` aktif.
> Frontend merender **hanya jabatan yang relevan** sesuai mode satuan kerja yang terinstal.

---

## 1. Hierarki Jabatan Resmi (Hasil Riset)

### A. Pengadilan Negeri (PN)

```
PIMPINAN
├── Ketua Pengadilan Negeri
└── Wakil Ketua Pengadilan Negeri

HAKIM
├── Hakim Madya Utama
├── Hakim Madya Muda
├── Hakim Madya Pertama
└── Hakim Pratama (dst)

KEPANITERAAN (dipimpin: Panitera)
├── Panitera Muda Perdata
├── Panitera Muda Pidana
├── Panitera Muda Hukum
├── Panitera Muda Tipikor      ← hanya IA Khusus/IA
├── Panitera Muda PHI          ← hanya kelas tertentu
├── Panitera Muda Niaga        ← hanya kelas tertentu
├── Panitera Muda Perikanan    ← hanya kelas tertentu
├── Panitera Pengganti (banyak orang)
└── Jurusita / Jurusita Pengganti

KESEKRETARIATAN (dipimpin: Sekretaris)
├── Sub Bagian Perencanaan, TI & Pelaporan
├── Sub Bagian Kepegawaian, Organisasi & Tata Laksana
└── Sub Bagian Umum & Keuangan

JABATAN FUNGSIONAL
├── Analis Perkara Peradilan
├── Pengelola Perkara
├── Pengadministrasi Hukum
├── Analis Akuntabilitas Kinerja Aparatur
├── Analis Perencanaan, Evaluasi & Pelaporan
├── Analis SDM Aparatur
├── Pranata Komputer
├── Arsiparis
└── Pengelola Sistem & Jaringan

PEGAWAI NON-ASN
└── PPNPN (Pegawai Pemerintah Non Pegawai Negeri)
```

### B. Pengadilan Tinggi (PT)

```
PIMPINAN
├── Ketua Pengadilan Tinggi
└── Wakil Ketua Pengadilan Tinggi

HAKIM TINGGI
└── Hakim Tinggi / Hakim Tinggi Pengawas Daerah

KEPANITERAAN (dipimpin: Panitera)
├── Panitera Muda Perdata
├── Panitera Muda Pidana
├── Panitera Muda Hukum
└── Panitera Pengganti

KESEKRETARIATAN (dipimpin: Sekretaris)
├── Bagian Perencanaan & Kepegawaian
│   ├── Sub Bagian Perencanaan, TI & Pelaporan
│   └── Sub Bagian Kepegawaian & Ortala
└── Bagian Umum & Keuangan
    ├── Sub Bagian Umum
    └── Sub Bagian Keuangan & Perbendaharaan

JABATAN FUNGSIONAL
├── Analis SDM Aparatur
├── Analis Pengelolaan Keuangan APBN
├── Arsiparis
└── Pranata Komputer

PEGAWAI NON-ASN
└── PPNPN
```

---

## 1B. Tabel Perbedaan Kritis PT vs PN

| Jabatan / Unit | PN | PT | Catatan |
|---|---|---|---|
| Ketua | Ketua PN | Ketua PT | Beda nomenklatur |
| Hakim | Hakim Madya/Pratama | **Hakim Tinggi** | Sangat berbeda |
| Kepaniteraan | Panitera Muda Perdata, Pidana, Hukum **+ Tipikor/PHI/Niaga/Perikanan (kondisional)** | Panitera Muda Perdata, Pidana, Hukum **saja** | PN lebih banyak divisi perkara |
| Jurusita | ✅ Ada | ❌ Tidak ada | Hanya di PN |
| Kesekretariatan | 3 Sub Bagian (flat) | 2 Bagian → masing-masing 2 Sub Bagian (berjenjang) | Struktur berbeda |
| Hakim Pengawas Daerah | ❌ | ✅ Ada | Hanya di PT |
| PPNPN | ✅ Ada | ✅ Ada | Sama |

### Jabatan Kondisional di PN (berdasarkan `court_class`):
| Jabatan | IA Khusus | IA | IB | II |
|---|---|---|---|---|
| Panitera Muda Tipikor | ✅ | ✅ | ❌ | ❌ |
| Panitera Muda PHI | ✅ | ✅ | ❌ | ❌ |
| Panitera Muda Niaga | ✅ | ❌ | ❌ | ❌ |
| Panitera Muda Perikanan | ✅ | ❌ | ❌ | ❌ |
| Panitera Muda Perdata | ✅ | ✅ | ✅ | ✅ |
| Panitera Muda Pidana | ✅ | ✅ | ✅ | ✅ |
| Panitera Muda Hukum | ✅ | ✅ | ✅ | ✅ |

## 2. Skema Database (Tambahan Domain 6)

### Mekanisme Dinamis PT vs PN:
```
Frontend/Admin request: GET /api/org/chart
    ↓
Backend baca global_configs → ambil tenant_mode + court_class
    ↓
Query filter org_positions:
  WHERE applicable_mode IN ('ALL', tenant_mode)
  AND (
    applicable_court_class IS NULL          ← berlaku semua kelas
    OR applicable_court_class @> court_class ← PostgreSQL JSONB contains
  )
    ↓
Hasilkan tree hanya jabatan yang relevan untuk satuan kerja ini
```

### Seed Data Awal (`bun run db:seed`):
- Isi `org_positions` SEKALI untuk **semua** jabatan (PT + PN + semua kelas)
- Tidak perlu konfigurasi jabatan ulang saat instal di satuan kerja baru
- Backend otomatis filter berdasarkan `global_configs` yang sudah diisi Admin

### Tabel Tambahan di Domain 6:

**`org_positions` Table** (Master Jabatan — di-seed lengkap PT+PN semua kelas)
- `id` (serial, pk)
- `position_code` (varchar, unique) → e.g. `KETUA_PN`, `HAKIM_TINGGI_PT`, `PANMUD_TIPIKOR_PN`
- `position_name` (varchar) → Nama resmi jabatan
- `position_category` (enum: `PIMPINAN`, `HAKIM`, `KEPANITERAAN`, `KESEKRETARIATAN`, `JABATAN_FUNGSIONAL`, `PPNPN`)
- `parent_position_id` (int, ref: org_positions.id, nullable) → Self-referencing hierarki
- `applicable_mode` (enum: `ALL`, `PT_ONLY`, `PN_ONLY`) → **Filter utama PT vs PN**
- `applicable_court_class` (jsonb, nullable) → `["IA_KHUSUS","IA"]` = hanya kelas tsb; `null` = semua kelas
- `is_structural` (boolean, default true) → Struktural vs fungsional
- `order_index` (int) → Urutan di bagan
- `is_active` (boolean, default true)

**`org_chart_nodes` Table** (Penempatan Aktual Pegawai ke Posisi)
- `id` (serial, pk)
- `staff_id` (int, ref: staff_members.id)
- `position_id` (int, ref: org_positions.id)
- `parent_node_id` (int, ref: org_chart_nodes.id, nullable) → Untuk tree renderer
- `effective_from` (date) → Tanggal mulai menjabat
- `effective_until` (date, nullable) → `null` = masih aktif
- `sk_number` (varchar, nullable) → Nomor SK pengangkatan
- `sk_date` (date, nullable)
- `sk_file_url` (varchar, nullable) → PDF SK jabatan
- `is_acting` (boolean, default false) → Pejabat Sementara (Plt./Plh.)
- `is_active` (boolean, default true)
- `created_at` (timestamp)

> **Arsitektur Data Ringkas**:
> - `staff_members` = data personal (nama, foto, NIP, riwayat)
> - `org_positions` = master jabatan seeded di awal (mencakup PT + PN + semua kelas)
> - `org_chart_nodes` = peta aktual (siapa menjabat apa, sejak kapan)
> - **Satu codebase, satu seed data — tampilan berbeda otomatis per satuan kerja**

> `org_chart_nodes` adalah data jabatan saat ini (siapa menjabat apa, sejak kapan).
> Keduanya dihubungkan untuk menghasilkan bagan organisasi yang akurat.

---

## 3. Spesifikasi Admin Panel

### Halaman: `/admin/struktur-organisasi`

#### Tab 1: Master Jabatan
- Tabel daftar semua `org_positions`
- Tombol Tambah Jabatan → form (nama, kategori, parent jabatan, mode PT/PN, kelas)
- Edit / Nonaktifkan jabatan (jabatan tidak dihapus agar riwayat terjaga)

#### Tab 2: Penempatan Aktual (Bagan Aktif)
- Tampilan **Tree View** interaktif (React Island: `<OrgChartEditor />`)
- Drag & drop untuk mengubah posisi atasan-bawahan
- Klik node → popup form: pilih staff, tanggal efektif, nomor SK
- Toggle "Plt./Plh." per posisi
- Tombol Export → unduh bagan sebagai PNG/PDF

#### Tab 3: Riwayat Jabatan per Pegawai
- Pilih nama pegawai → tampil timeline jabatan-jabatan yang pernah dipegang
- Berguna untuk data karier di halaman profil pegawai

---

## 4. Tampilan Frontend (Halaman Publik)

### Halaman: `/tentang/struktur-organisasi`

#### Mode Tampilan (dapat dikonfigurasi Admin):

**A. Bagan Hierarki Visual (org-chart tree)**
- Render pohon jabatan dari atas (Ketua) hingga bawah
- Setiap node berisi: Foto + Nama + Jabatan
- Hover → card popup dengan detail singkat pegawai
- Komponen React Island: `<OrgChartViewer />`
- Responsive: di mobile, tree diganti menjadi accordion berlapis

**B. Tampilan Per Divisi (Grid Foto)**
- Kelompokkan per kategori: Pimpinan, Hakim, Kepaniteraan, Kesekretariatan
- Setiap kelompok tampil sebagai card grid foto pegawai
- Klik foto → modal detail (pendidikan, riwayat karier)

#### Data yang Ditampilkan per Node:
- Foto resmi (dari `staff_members.photo_url`)
- Nama lengkap
- Jabatan (dari `org_positions.position_name`)
- Status Plt./Plh. jika `is_acting = true` → tampilkan badge "Plt."
- NIP (opsional, tergantung setting admin)

#### Conditional Rendering (sesuai `court_class`):
- Panitera Muda Tipikor → hanya tampil jika `court_class = IA_KHUSUS || IA`
- Panitera Muda PHI, Niaga, Perikanan → hanya tampil jika kelas sesuai
- Hakim Tinggi → hanya tampil jika mode = `PENGADILAN_TINGGI`

---

## 5. Komponen React Island Baru

```tsx
<OrgChartEditor />
// Admin Panel: Tree editor drag & drop untuk penempatan jabatan
// Props: positions, staffMembers, currentNodes
// Events: onNodeUpdate, onNodeAdd, onNodeRemove

<OrgChartViewer />
// Frontend Publik: Bagan org read-only dengan hover detail
// Props: nodes, displayMode ('tree' | 'grid')
// Responsive: tree di desktop, accordion di mobile
```

---

## 6. API Endpoint Baru (Backend)

```
GET  /api/org/positions          → Master daftar jabatan (filter: mode, kelas, kategori)
POST /api/org/positions          → Tambah jabatan baru
PATCH /api/org/positions/:id     → Edit jabatan

GET  /api/org/chart              → Tree nodes aktif (untuk Frontend & Admin viewer)
POST /api/org/chart              → Tambah penempatan (staff → posisi)
PATCH /api/org/chart/:id         → Update penempatan (ganti pegawai, status Plt.)
DELETE /api/org/chart/:id        → Nonaktifkan penempatan (set effective_until)

GET  /api/org/history/:staff_id  → Riwayat jabatan pegawai
```

---

*Catatan: Fitur ini menggantikan dan memperluas tabel `staff_members` yang ada. Data personal tetap di `staff_members`, jabatan struktural di `org_positions`, dan penempatan aktual di `org_chart_nodes`.*
