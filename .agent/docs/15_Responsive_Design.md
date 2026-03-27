# 15. Standar Responsive Design (Multi-Device)

Sesuai Juknis SK DJU 127/2026, website **wajib optimal** di seluruh perangkat. Dokumen ini adalah panduan teknis implementasi responsivitas menggunakan **Tailwind CSS Breakpoint System** di dalam ekosistem Astro + Shadcn/UI.

---

## 1. Breakpoint System (Tailwind CSS)

| Breakpoint | Prefix Tailwind | Lebar Layar | Target Perangkat |
|---|---|---|---|
| Mobile S | *(default)* | 0 — 479px | Smartphone kecil (HP Android entry-level) |
| Mobile L | `sm:` | 480px — 767px | Smartphone besar / iPhone |
| Tablet | `md:` | 768px — 1023px | iPad, tablet Android |
| Laptop | `lg:` | 1024px — 1279px | Laptop 13-14 inch |
| Desktop | `xl:` | 1280px — 1535px | Monitor standar |
| Wide | `2xl:` | 1536px ke atas | Monitor lebar / layar TV gedung pengadilan |

> **Prinsip**: Kembangkan dengan pendekatan **Mobile First** — tulis style default untuk mobile, lalu tambahkan override untuk layar lebih besar.

---

## 2. Perilaku Layout per Breakpoint

### Header
| Elemen | Mobile (< 768px) | Tablet (768-1023px) | Desktop (≥ 1024px) |
|---|---|---|---|
| Logo | Kecil, kiri atas | Sedang | Penuh |
| Nama Pengadilan | Disembunyikan atau dipersingkat | 1 baris singkat | 2 baris penuh |
| Badge WBK/WBBM | Disembunyikan | Tampil kecil | Tampil penuh |
| Navigasi | Hamburger menu (☰) → Drawer | Hamburger atau nav ringkas | Full horizontal nav bar |

### Navigation Bar
- **Mobile**: Menu nav disembunyikan, diganti ikon **hamburger (☰)**. Saat diklik, muncul **Drawer/Sidebar** dari kiri yang menampilkan semua menu + sub-menu bertingkat.
- **Tablet**: Bisa menggunakan nav horizontal terpotong atau hamburger.
- **Desktop**: Full horizontal navbar 7 menu wajib, sub-menu tampil sebagai **dropdown hover**.

### Hero Section
| Elemen | Mobile | Tablet | Desktop |
|---|---|---|---|
| Slider Banner | Tinggi dikurangi (200px) | Tinggi sedang (350px) | Tinggi penuh (450-500px) |
| Overlay Teks | Font lebih kecil, 1 baris | Font sedang | Font besar, 2 baris |
| Quick Links Grid | **2 kolom x 5 baris** (10 icon vertikal) | **3-4 kolom** | **5 kolom x 2 baris** (layout Juknis) |

### Body Layout (Konten Utama + Sidebar)
| Layout | Mobile | Tablet | Desktop |
|---|---|---|---|
| Kolom | **1 kolom penuh** (sidebar pindah ke bawah) | **1 kolom** atau sidebar accordion | **2 kolom**: Kiri 70% + Kanan 30% |
| Berita Grid | 1 kolom | 2 kolom | 3 kolom |
| RSS Feed | Stack vertikal | 2 kolom | 2 kolom (sesuai wireframe Juknis) |

### Admin Panel
| Komponen | Mobile | Tablet | Desktop |
|---|---|---|---|
| Sidebar Admin | Icon-only + Drawer overlay | Collapsible sidebar | Full sidebar 250px |
| Tabel Data (Shadcn) | Horizontal scroll + responsive columns | Sebagian kolom | Full tabel |
| Form Editor | Single column | Single column | 2-column layout |

---

## 3. Komponen Khusus Mobile

### Hamburger Menu (React Island)
```tsx
// Komponen: <MobileNav />
// Perilaku:
// - Tombol ☰ muncul hanya di < md breakpoint (md:hidden)
// - Drawer dari kiri dengan semua menu + submenu accordion
// - Overlay hitam semi-transparan saat drawer terbuka
// - Tutup otomatis saat klik di luar area drawer
// - Smooth transition 300ms
```

### Slider Mobile
- Swipe gesture support (touch event) menggunakan library ringan (Swiper.js atau native CSS scroll-snap)
- Pagination dots di bawah slider (bukan arrow) untuk layar kecil
- Auto-play dengan interval 5 detik, pause saat disentuh

### Quick Links Mobile (Grid Adaptif)
```
Mobile  : grid-cols-2  → 2 kolom, 5 baris
Tablet  : grid-cols-4  → 4 kolom, 3 baris
Desktop : grid-cols-5  → 5 kolom, 2 baris (Juknis wireframe)
```

---

## 4. Aturan Gambar Responsif

- Semua gambar menggunakan komponen `<Image />` dari Astro (`astro:assets`)
- Otomatis generate multiple ukuran: `400w`, `800w`, `1200w`
- Atribut `sizes` wajib didefinisikan: `sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"`
- Format output: `.webp` (default) dengan fallback `.jpg`
- `loading="lazy"` untuk semua gambar di bawah fold
- `loading="eager"` hanya untuk hero banner pertama (LCP optimization)

---

## 5. Typography Responsif

```css
/* Skala tipografi (implementasi via Tailwind) */
Heading H1: text-2xl  (mobile) → text-4xl  (desktop)
Heading H2: text-xl   (mobile) → text-3xl  (desktop)
Heading H3: text-lg   (mobile) → text-2xl  (desktop)
Body Text:  text-sm   (mobile) → text-base (desktop)
Caption:    text-xs   (mobile) → text-sm   (desktop)
```

---

## 6. Touch & Interaction

- **Touch Target**: Semua tombol dan link minimum **44x44px** (WCAG 2.5.5)
- **Tap Delay**: Tidak ada delay 300ms — gunakan `touch-action: manipulation`
- **Hover State**: Style hover hanya diterapkan pada perangkat non-touch (`@media (hover: hover)`)
- **Swipe Gesture**: Slider hero mendukung swipe kiri/kanan di mobile
- **Scroll**: Halaman tidak ada horizontal scroll di mobile (overflow-x: hidden)

---

## 7. Testing Responsif (Wajib Sebelum Deploy)

### Perangkat yang Harus Diuji:
| Perangkat | Resolusi | Prioritas |
|---|---|---|
| iPhone SE / Android entry-level | 375 x 667px | 🔴 Wajib |
| iPhone 14 / Samsung Galaxy A | 390 x 844px | 🔴 Wajib |
| iPad / Tablet Android | 768 x 1024px | 🟡 Penting |
| Laptop 13 inch | 1280 x 800px | 🔴 Wajib |
| Monitor Full HD | 1920 x 1080px | 🟡 Penting |

### Tools Testing:
```bash
# Chrome DevTools → Toggle Device Toolbar (Ctrl+Shift+M)
# Lighthouse Mobile audit → gunakan mode "Mobile" bukan "Desktop"
# Real device test: minimal iPhone + Android entry-level
```

### Lighthouse Target (Mode Mobile):
- Performance ≥ 90
- Accessibility: 100
- Best Practices: 100
- SEO: 100

---

## 8. Implementasi di Tailwind (Contoh Pola Nyata)

```astro
<!-- Layout 2-kolom yang collapse ke 1-kolom di mobile -->
<div class="grid grid-cols-1 lg:grid-cols-[70%_30%] gap-6">
  <main><!-- Konten Utama --></main>
  <aside><!-- Sidebar --></aside>
</div>

<!-- Quick Links Grid Adaptif -->
<div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 xl:grid-cols-5 gap-4">
  <!-- 10 icon layanan -->
</div>

<!-- Nav: Desktop full, Mobile hamburger -->
<nav class="hidden md:flex gap-6"><!-- Full Nav --></nav>
<button class="md:hidden">☰</button><!-- Hamburger -->
```

---

*Catatan Bagi Agen/Developer: Dokumen ini adalah panduan wajib saat membangun setiap komponen. Cek selalu bahwa komponen yang dibuat sudah diuji di breakpoint mobile (375px) dan desktop (1280px) minimal.*
