# 05. Project Folder Structure (Decoupled Monorepo)

Struktur monorepo menggunakan **Bun Workspaces** adalah landasan The Speed Stack. Struktur ini menjamin Frontend (Astro) memiliki persepsi yang sama terhadap struktur data Backend (Elysia).

```text
/ (Project Root)
├── package.json              # Definisi bun workspaces utama
├── bun.lockb                 # Binary lockfile Bun yang super cepat (Wajib dicommit)
├── apps/
│   ├── api-server/           # BACKEND (Bun + Elysia + Drizzle)
│   │   ├── src/
│   │   │   ├── db/           # Koneksi Postgres & Drizzle Migrations (`schema.ts`)
│   │   │   ├── plugins/      # Security, CORS, Rate Limiters
│   │   │   ├── routes/       # Endpoint API (User, Berita, Layouts)
│   │   │   └── index.ts      # Server Bootstrap & Eden Type Exporter
│   │   ├── package.json
│   │   └── .env
│   │
│   └── web-portal/           # FRONTEND (Astro + React Islands + Shadcn/UI)
│       ├── public/           # Aset statis terbuka
│       ├── src/
│       │   ├── components/   
│       │   │   ├── ui/       # Komponen generik dari Shadcn (Button, Dialog, Table)
│       │   │   └── blocks/   # Komponen gabungan spesifik (Header, Jadwal Sidang)
│       │   ├── layouts/      # Induk desain dasar (Standar Layout Juknis)
│       │   ├── pages/        # File .astro (Routing halaman web utama)
│       │   └── lib/          # Helper, Eden Connector fetcher, utils Shadcn
│       ├── package.json
│       ├── astro.config.mjs
│       ├── tailwind.config.mjs # Registrasi styling
│       └── .env
│
├── packages/
│   └── shared-types/         # SKEMA DEKLARATIF BERSAMA
│       ├── src/
│       │   └── index.ts      # Berisi schema Zod/TypeBox yang di-share
│       └── package.json
│
├── infra/                    # IaC (Infrastructure as Code)
│   ├── docker/               # Dockerfile multi-stage untuk apps/api-server & web-portal
│   └── nginx/                # Konfigurasi reverse proxy
└── README.md
```

## Aturan Workspace
- Tidak ada instalasi dependensi silang menggunakan `npm/yarn/pnpm`. Cukup jalankan `bun install` dari root folder. Bun akan memetakan symlink ke setiap struktur.
- Dependensi internal dipanggil memakai `workspace:*` di package.json masing-masing sub-direktori.
- Pemasangan Shadcn UI diarahkan menggunakan direktori `src/components/ui/` melalui *Astro CLI integration*.
