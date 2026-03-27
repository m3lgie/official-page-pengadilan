# 09. Development Rules & Coding Standards

Semua pemrograman (Astro, Bun, Drizzle Monorepo) untuk portal Pengadilan Tinggi dan Negeri ini wajib menundukkan kerangka berikut demi kolaborasi yang tangguh.

## 1. General Rules & API Tenancy

- **Single Codebase Tenancy**: Tidak ada *hardcode* teks "Tipikor" atau "Pengadilan Tinggi" atau "Negeri" pada `apps/web-portal`. Front-End wajib merender dinamis berdasarkan Payload JSON penyebutan CMS.
- **No Placeholders**: Dilarang menggunakan teks "Lorem Ipsum" atau gambar placeholder generic. Wajib gunakan data simulasi yang benar-benar mensimulasikan hukum Indonesia.
- **Security First**: Semua input interaktif masyarakat maupun CMS wajib divalidasi dan di-sanitize ganda oleh *TypeBox* Bun. Celah `isInternalAdmin` harus ditutup erat.
- **Documentation**: Setiap fungsi *React Island* kompleks atau rute Elysia wajib memiliki dokumentasi JSDoc param & return.

## 2. Git Workflow

- **Branching**: `main` (Production Docker), `develop` (Staging/Lokal), `feature/xyz` (Development spesifik).
- **Commit Message**: Mengikuti standar absolut [Conventional Commits](https://www.conventionalcommits.org/). Tidak menjejalkan 3 fitur dalam 1 kalimat *commit*. Perubahan di `root` Workspace Bun dicatat rapi.

## 3. Accessibility Enforcement

- Lolos total evaluasi *WCAG* pada tes rutin (Misal `axe-core`).
- Segala elemen `<Image />` Astro yang berisi icon atau pamflet Wajib memiliki `aria-label` / *Alt text*. Semua elemen interaktif publik (seperti navigasi difabel) disematkan atribut tersebut.

## 4. Performance & SEO Standards (The Speed Stack Goal)

- **Lighthouse Score**: Target perolehan 100/100 pada Performance, Accessibility, dan SEO berkat *Zero-JS Output* Engine Astro.
- **Image Optimization**: Tidak ada PNG / JPG mentah bertengger di produksi. Semua internal dibabat ekstensi mutlak `.webp` oleh optimasi *astro:assets*.
- **Semantic HTML**: Pemakaian tag semantik ketat (`<article>`, `<section>`, `<nav>`, `<aside>`, dan penyuntikan JSON-LD / Meta OpenGraph dinamis dari *Server-Side Rendering*).
