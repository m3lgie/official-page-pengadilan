# 19. Standar Modularisasi Kode & Clean Architecture

Dokumen ini adalah **hukum ketat penulisan kode** untuk mencegah terbentuknya file raksasa ("Spaghetti Code" / Monolithic Script) yang menelan ribuan baris. Prinsip dasarnya: **"Pisahkan Logika, Pecahkan Kompleksitas"**.

Tidak boleh ada _script_ (baik Astro, React, maupun Elysia) yang melampaui **300-400 baris kode murni**. Jika terjadi, fungsi tersebut DIPASTIKAN melanggar prinsip _Single Responsibility_ dan wajib direfaktor ke dalam modul/helai file baru.

---

## 1. Modularisasi Backend (ElysiaJS + Bun)

Backend dilarang meletakkan definisi _Route_, Validasi (_TypeBox_), dan _Business Logic_ (seperti pengolahan database) ke dalam satu file rute yang sama.

Arsitektur wajib `apps/api-server/src/`:

- `routes/` — **Hanya definisi Endpoint & Validasi (TypeBox)**. File ini hanya menerima request, memanggil `services`, dan mengembalikan response.
- `services/` — **Logika Bisnis**. Tempat pemrosesan data, kalkulasi, pemanggilan API eksternal, dan struktur if/else rumit.
- `db/queries/` — **Akses Data Tunggal**. Fungsi Drizzle ORM tunggal berefisiensi tinggi. Rute/Service memanggil kueri dari sini.

  **Contoh Eksekusi Elysia yang Tepat (Elysia Plugins):**
  Gunakan `app.use()` untuk memecah rute per domain (misal: `app.use(postsRoute)`, `app.use(authRoute)`).

---

## 2. Pemecahan Skema Basis Data (Drizzle ORM)

Mengingat Sistem Pengadilan 2026 ini memiliki **14 Domain & 50+ Tabel**, MENYATUKANNYA ke dalam satu file raksasa `src/db/schema.ts` adalah sebuah malapetaka visibilitas.

- Skema wajib digunting berdasarkan domainnya masing-masing di dalam folder `src/db/schema/`.
- **Struktur Pemecahan:**
  ```text
  src/db/schema/
   ├── 01_auth.ts          (Tabel users, accounts)
   ├── 02_config.ts        (Tabel global_configs)
   ├── 03_cms.ts           (Tabel posts, categories)
   ├── ...
   └── index.ts            ← HANYA MELAKUKAN экспорт (export * from './..')
  ```
- File relasi (_relations_) Drizzle wajib didefinisikan secara intim di file yang memuat tabel berelasi tersebut.

---

## 3. Modularisasi Frontend Publik (Astro Zero-JS)

Membuat halaman `index.astro` dengan 1000 baris HTML karena seluruh blok layout (Berita, Headline, Jadwal Sidang) dituangkan di situ adalah **dilarang**.

- Halaman Induk (Contoh `pages/index.astro`) harus sesederhana mungkin, hanya menjadi perakit / lem komponen.
- **Pemisahan Astro Components:**
  ```astro
  ---
  import BaseLayout from '@/layouts/BaseLayout.astro';
  import HeroSlider from '@/components/home/HeroSlider.astro';
  import QuickLinks from '@/components/home/QuickLinks.astro';
  import NewsGrid from '@/components/home/NewsGrid.astro';
  ---
  <BaseLayout title="Beranda | Pengadilan">
      <HeroSlider />
      <QuickLinks />
      <NewsGrid />
  </BaseLayout>
  ```
- **Max Lines per Component**: Batasi di bawah 250 baris. Jika sebuah UI Block menjadi rumit, belah lagi menjadi _Sub-component_ (e.g. `NewsCard.astro`).

---

## 4. Efisiensi & Keringkasan React Islands (Admin Panel)

Komponen berlabel cerdas `client:load` di dalam folder `apps/web-portal/src/components/admin/` rentan melendung jika _State Management_ tak terawat dan _Custom Hooks_ dibiarkan campur aduk dengan _Markup Render (TSX)_.

- Jika sebuah _Component_ memiliki logika _fetching_ data panjang, pisah fungsi fetch-nya ke sebuah _Custom Hook_ murni: `hooks/usePosts.ts`.
- Pisahkan presentasi _(View)_ dari Logika Bisnis (misal: penanganan _Drag & Drop_ tata letak menggunakan dnd-kit) dengan pola presentasi berlapis.
- **Shadcn/UI components** (Button, Dialog, Togel, Input) telah dipisah ke dalam modul `components/ui`. Gunakan _reusable components_ ini; dilarang membangun tombol murni bawaan HTML `<button class="...">` dari awal di halaman admin jika varian _Shadcn_ sudah tersedia.
