# Nubi Academy — Frontend

Platform kursus online **Nubi Academy** — *"Learn Anything in AI Era"*.
Frontend berbasis **Next.js 15 (App Router)** + **React 19** + **TypeScript**, terhubung ke backend **Payload CMS** dan pembayaran **Midtrans**.

Repo: `https://github.com/bbal10/nubiacademy.git`
Branch utama: `master` (pengembangan aktif di `dev`)

---

## Daftar Isi

- [Fitur](#fitur)
- [Tech Stack](#tech-stack)
- [Prasyarat](#prasyarat)
- [Mulai Cepat](#mulai-cepat)
- [Environment Variables](#environment-variables)
- [Struktur Proyek](#struktur-proyek)
- [Routing & Halaman](#routing--halaman)
- [API Routes (BFF)](#api-routes-bff)
- [Autentikasi](#autentikasi)
- [State, Data & Cache Lokal](#state-data--cache-lokal)
- [Pembayaran](#pembayaran)
- [Styling & UI](#styling--ui)
- [Script NPM](#script-npm)
- [Build & Deploy](#build--deploy)
- [Konvensi Git](#konvensi-git)
- [Troubleshooting](#troubleshooting)

---

## Fitur

- **Landing page** publik: hero, marquee kategori, fitur, komunitas, footer.
- **Katalog & detail kelas**: list kelas, detail per `slug`, halaman checkout per kelas.
- **Auth**: daftar, masuk, proteksi route via `middleware.ts` (cookie `payload-token`).
- **Dashboard siswa** (terproteksi):
  - `/dashboard/kelas` — kelas saya, ongoing, selesai, pinned.
  - `/dashboard/kelas/[slug]` & `/dashboard/kelas/[slug]/[id]` — materi / lesson.
  - `/dashboard/akun-saya` — profil.
- **Order & kupon**: pembuatan order ke Payload CMS, validasi kupon via `/api/coupon/[code]`.
- **Pembayaran Midtrans**: halaman sukses `/payment/success`.
- **UX**: dark/light mode (`next-themes`), toast (`sonner` + Radix Toast), carousel, chart (`recharts`), video player (`react-player`).
- **Offline-friendly parsial**: cache user via IndexedDB (`dexie`).

---

## Tech Stack

| Layer | Teknologi |
|---|---|
| Framework | Next.js `15.0.4` (App Router, Turbopack dev) |
| UI Library | React `19`, React DOM `19` |
| Bahasa | TypeScript `5` |
| Styling | Tailwind CSS `3.4`, `tailwindcss-animate`, `@tailwindcss/typography`, `tailwind-scrollbar-hide` |
| Komponen | Radix UI (accordion, avatar, checkbox, collapsible, dialog, label, navigation-menu, progress, separator, slot, tabs, toast, tooltip), `class-variance-authority`, `clsx`, `tailwind-merge`, `vaul`, `lucide-react` |
| Form | `react-hook-form`, `@hookform/resolvers`, `zod` |
| Data fetching | `axios`, `@tanstack/react-query` v4 |
| State | `zustand` (`store/useKelasPositionStore.ts`), React Context (`context/`) |
| Auth / session | `cookies-next`, `jose`, `jsonwebtoken` (verifikasi JWT Payload di server) |
| Storage lokal | `dexie` + `dexie-react-hooks` (IndexedDB `nubiacademy-db`) |
| Backend CMS | Payload CMS (diakses via `NEXT_PUBLIC_MAIN_ENDPOINT`) |
| Payment | `midtrans-client` |
| Media / misc | `embla-carousel-react`, `@devnomic/marquee`, `react-player`, `recharts`, `react-tooltip`, `uuid`, `sonner`, `next-themes` |

Lihat lengkap di [`package.json`](./package.json).

---

## Prasyarat

- **Node.js** `>= 20` (disarankan LTS terbaru)
- **npm** `>= 10` (repo ini memakai `package-lock.json`)
- Backend **Payload CMS** yang sudah berjalan (untuk endpoint users, kelas, orders, kupon)
- Akun / kredensial **Midtrans** (jika menguji pembayaran end-to-end)

Cek versi:

```bash
node -v
npm -v
```

---

## Mulai Cepat

```bash
# 1. Clone
git clone https://github.com/bbal10/nubiacademy.git
cd nubiacademy

# 2. Install dependencies
npm install

# 3. Siapkan environment
cp .env.example .env.local
# lalu isi variabel di bawah

# 4. Jalankan dev server (Turbopack)
npm run dev

# 5. Buka
# http://localhost:3000
```

> Dev server memakai `next dev --turbopack` (lihat `package.json`).

---

## Environment Variables

Buat file `.env.local` di root (file `.env*` sudah di-ignore Git, lihat [`.gitignore`](./.gitignore)).

| Variabel | Wajib | Dipakai di | Keterangan |
|---|---|---|---|
| `NEXT_PUBLIC_MAIN_ENDPOINT` | Ya | `services/global.ts` (`instance`) | Base URL API Payload CMS, mis. `https://api.nubiacademy.id/api` |
| `NEXT_PUBLIC_LOCAL_ENDPOINT` | Ya | `services/global.ts` (`local`) | Base URL BFF / API internal Next.js, mis. `http://localhost:3000/api` |
| `LOCAL_ENDPOINT` | Ya (server) | `app/kelas/[slug]/page.tsx` | Base URL server-side untuk fetch kelas (tanpa prefix `NEXT_PUBLIC_`) |
| `PAYLOAD_SECRET` | Ya (server) | `lib/server.ts` | Secret Payload CMS untuk verifikasi JWT (`HS256`, di-hash SHA-256 lalu dipotong 32 char) |

Contoh `.env.example` / `.env.local`:

```env
# Backend Payload CMS
NEXT_PUBLIC_MAIN_ENDPOINT=https://cms-anda.up.railway.app/api

# BFF Next.js (route handlers di /app/api)
NEXT_PUBLIC_LOCAL_ENDPOINT=http://localhost:3000/api
LOCAL_ENDPOINT=http://localhost:3000/api

# Harus sama dengan PAYLOAD_SECRET di backend CMS
PAYLOAD_SECRET=isi-dengan-secret-yang-sama-seperti-backend
```

> `midtrans-client` terdaftar sebagai dependency untuk sisi pembayaran. Jika ada server key / client key Midtrans, tambahkan sebagai variabel baru (mis. `MIDTRANS_SERVER_KEY`, `NEXT_PUBLIC_MIDTRANS_CLIENT_KEY`) dan jangan commit ke Git.

---

## Struktur Proyek

```
nubiacademy/
├── app/                        # App Router (pages, layouts, API routes)
│   ├── page.tsx                # Landing page (Navbar, Hero, Marquee, Features, Community, Footer)
│   ├── layout.tsx              # Root layout (font Plus Jakarta Sans, Theme + React Query providers)
│   ├── globals.css
│   ├── masuk/page.tsx          # Login
│   ├── daftar/page.tsx         # Register
│   ├── kelas/
│   │   ├── page.tsx            # Katalog kelas
│   │   └── [slug]/
│   │       ├── page.tsx        # Detail kelas
│   │       └── checkout/page.tsx
│   ├── dashboard/              # Area terproteksi (middleware)
│   │   ├── page.tsx
│   │   ├── kelas/page.tsx
│   │   ├── kelas/[slug]/page.tsx
│   │   ├── kelas/[slug]/[id]/page.tsx
│   │   └── akun-saya/page.tsx
│   ├── payment/success/page.tsx
│   └── api/                    # BFF / route handlers (proxy + validasi ke Payload CMS)
│       ├── login/route.ts
│       ├── me/route.ts
│       ├── me/profile/route.ts
│       ├── kelas/route.ts
│       ├── kelas/[slug]/route.ts
│       ├── kelas/me/route.ts
│       ├── kelas/me/ongoing/route.ts
│       ├── kelas/me/done/route.ts
│       ├── kelas/pinned/route.ts
│       ├── lesson/[id]/route.ts
│       ├── coupon/[code]/route.ts
│       └── order/route.ts
├── components/
│   ├── layout/                 # navbar, app-sidebar, sections (hero, features, footer, ...), providers
│   └── ui/                     # shadcn-style primitives (button, dialog, form, ...)
├── components.json             # Konfigurasi shadcn/ui
├── constant/dummy.ts           # Data dummy / konstanta
├── context/
│   ├── authContext.tsx         # AuthStatus (YES/NO) berbasis cookie payload-token
│   └── courseContext.tsx       # Context kursus
├── hooks/                      # use-auth, use-course, use-lesson, use-mobile, use-toast
├── lib/
│   ├── server.ts               # checkCookieAndValidate() + performAction() (verifikasi JWT)
│   └── utils.ts                # cn() dkk.
├── services/global.ts          # axios instance (backend CMS) & local (BFF)
├── store/useKelasPositionStore.ts  # zustand store posisi kelas
├── util/                       # isValidMongoId, numberToKFormatter
├── db.ts                       # Dexie IndexedDB (tabel user)
├── middleware.ts               # Proteksi /dashboard & /checkout, redirect /masuk & /daftar
├── next.config.ts              # remotePatterns images: placehold.co, localhost, avatar.iran.liara.run
├── tailwind.config.ts
├── postcss.config.mjs
├── tsconfig.json               # alias @/* -> ./*
└── public/                     # Aset statis
```

---

## Routing & Halaman

| Route | Akses | Keterangan |
|---|---|---|
| `/` | Publik | Landing page |
| `/kelas` | Publik | Katalog kelas |
| `/kelas/[slug]` | Publik | Detail kelas |
| `/kelas/[slug]/checkout` | Login required (middleware, matcher akhiran `/checkout`) | Checkout kelas |
| `/masuk`, `/daftar` | Guest only (redirect ke dashboard jika sudah login) | Auth pages |
| `/dashboard` | Login required (redirect ke `/dashboard/kelas?position=kelas-saya`) | Index dashboard |
| `/dashboard/kelas` | Login required | Daftar kelas saya + query `?position=` |
| `/dashboard/kelas/[slug]` | Login required | Detail kelas yang diambil |
| `/dashboard/kelas/[slug]/[id]` | Login required | Lesson / materi |
| `/dashboard/akun-saya` | Login required | Profil |
| `/payment/success` | Publik (halaman status) | Konfirmasi pembayaran berhasil |

Logika proteksi ada di [`middleware.ts`](./middleware.ts) berbasis cookie `payload-token`.

---

## API Routes (BFF)

Semua di bawah `/app/api`, berperan sebagai proxy yang meneruskan request ke Payload CMS dengan header auth yang benar:

| Method & Path | Fungsi |
|---|---|
| `POST /api/login` | Login ke `{CMS}/users/login`, teruskan `Set-Cookie` ke browser |
| `GET /api/me` | Ambil user saat ini (validasi cookie) |
| `GET/PATCH /api/me/profile` | Baca / update profil |
| `GET /api/kelas` | List kelas |
| `GET /api/kelas/[slug]` | Detail kelas |
| `GET /api/kelas/me` | Kelas milik user |
| `GET /api/kelas/me/ongoing` | Kelas sedang berjalan |
| `GET /api/kelas/me/done` | Kelas selesai |
| `GET /api/kelas/pinned` | Kelas yang di-pin |
| `GET /api/lesson/[id]` | Detail lesson |
| `GET /api/coupon/[code]` | Validasi kupon |
| `POST /api/order` | Buat order (`course_item`, `order_number`, `coupon_code`) ke `{CMS}/orders` via `multipart/form-data` + header `Authorization: JWT <payload-token>` |

Contoh alur order (`app/api/order/route.ts`):

```ts
// 1. Validasi JWT dari cookie payload-token (lib/server.ts)
// 2. Bungkus body ke FormData field "_payload"
// 3. POST ke {NEXT_PUBLIC_MAIN_ENDPOINT}/orders dengan Authorization: JWT <token>
```

---

## Autentikasi

1. User submit email + password ke `POST /api/login`.
2. Route handler meneruskan ke `POST {CMS}/users/login`.
3. Jika sukses, `Set-Cookie` dari CMS diteruskan ke browser (`payload-token`, `HttpOnly` dari sisi CMS).
4. Request berikutnya yang butuh auth divalidasi di server via `checkCookieAndValidate()` di [`lib/server.ts`](./lib/server.ts):
   - Ambil cookie `payload-token`.
   - Hash `PAYLOAD_SECRET` dengan SHA-256, potong 32 char sebagai key.
   - `jwt.verify(token, key, { algorithms: ["HS256"] })`.
5. `middleware.ts` menjaga route:
   - Belum login + akses `/dashboard*` atau `*/checkout` → redirect `/masuk` (sambil menyimpan `redirectUrl` di cookie).
   - Sudah login + akses `/masuk` / `/daftar` → redirect ke dashboard.
6. Sisi client, `AuthContext` (`context/authContext.tsx`) membaca keberadaan cookie untuk toggle UI login/logout.

---

## State, Data & Cache Lokal

- **React Query** (`@tanstack/react-query` v4) via `ReactQueryProvider` di root layout untuk fetching + caching server state. Custom hooks: `hooks/use-auth.tsx`, `hooks/use-course.ts`, `hooks/use-lesson.ts`.
- **Zustand** (`store/useKelasPositionStore.ts`) untuk state ringan posisi/tab kelas di dashboard.
- **Context** (`context/authContext.tsx`, `context/courseContext.tsx`) untuk auth status & data kursus lintas komponen.
- **Dexie (IndexedDB)** (`db.ts`, database `nubiacademy-db`, tabel `user`) untuk cache profil user di browser.
- **Axios instances** (`services/global.ts`): `instance` → CMS, `local` → BFF Next.js.

---

## Pembayaran

- Pembuatan order via `POST /api/order`.
- Integrasi Midtrans memakai paket `midtrans-client` (snap / core API — sesuaikan dengan implementasi checkout di `app/kelas/[slug]/checkout`).
- Setelah pembayaran sukses, user diarahkan ke `/payment/success`.

> Untuk testing lokal Midtrans, gunakan sandbox keys dan pastikan webhook / redirect URL mengarah ke URL yang dapat diakses (gunakan tunnel seperti ngrok jika perlu).

---

## Styling & UI

- Tailwind CSS + `tailwindcss-animate` + `@tailwindcss/typography`.
- Font: **Plus Jakarta Sans** via `next/font/google` (`app/layout.tsx`).
- Theming: `next-themes` (`class` mode, `ThemeProvider` + `toggle-theme.tsx`).
- Primitif UI gaya shadcn di `components/ui` (lihat `components.json` untuk alias & konfigurasi).
- Image remote yang diizinkan (`next.config.ts`): `placehold.co`, `localhost`, `avatar.iran.liara.run`. Tambahkan hostname CMS/prod di sini bila thumbnail kelas di-load via `next/image`.

---

## Script NPM

| Script | Perintah | Keterangan |
|---|---|---|
| Dev | `npm run dev` | `next dev --turbopack` |
| Build | `npm run build` | Build produksi |
| Start | `npm run start` | Jalankan hasil build |
| Lint | `npm run lint` | `next lint` |

---

## Build & Deploy

```bash
npm run build
npm run start
```

- Output Next.js standar (`.next/` di-ignore, lihat `.gitignore`).
- Bisa di-deploy ke **Vercel**, VPS (PM2/Docker), atau platform Node.js lain.
- Checklist sebelum deploy:
  1. Set semua env di atas di dashboard hosting (gunakan URL produksi untuk `NEXT_PUBLIC_*`).
  2. Tambahkan hostname image produksi ke `images.remotePatterns` di `next.config.ts`.
  3. Pastikan backend Payload CMS mengizinkan CORS/credentials dari domain frontend.
  4. Pastikan `PAYLOAD_SECRET` sama persis antara CMS & frontend.

---

## Konvensi Git

Branch yang ada:

- `master` — stabil / rilis
- `dev` — pengembangan aktif

Alur yang disarankan:

```bash
# Ambil terbaru
git fetch origin
git checkout dev
git pull origin dev

# Buat branch fitur dari dev
git checkout -b feat/nama-fitur

# ... kerjakan, lalu commit ...
git add -p
git commit -m "feat: deskripsi singkat perubahan"

# Push & buka PR ke dev
git push -u origin feat/nama-fitur
```

- Gunakan [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `style:`.
- Jangan commit `.env*`, `node_modules/`, `.next/` (sudah di-ignore).
- Lakukan PR `feat/*` → `dev`, lalu `dev` → `master` saat rilis.

---

## Troubleshooting

| Gejala | Kemungkinan penyebab & solusi |
|---|---|
| `401 Token tidak ditemukan` | Cookie `payload-token` hilang / belum login. Login ulang via `/masuk`. Cek domain cookie CMS vs frontend. |
| `Token gagal direverifikasi` | `PAYLOAD_SECRET` frontend ≠ backend. Samakan nilainya. |
| Kelas/thumbnail tidak tampil | Host image belum ada di `images.remotePatterns` (`next.config.ts`). Tambahkan hostname CMS. |
| CORS / `withCredentials` gagal | Backend belum allow origin frontend + `credentials: true`. Periksa juga `NEXT_PUBLIC_MAIN_ENDPOINT`. |
| Loop redirect `/masuk` ↔ `/dashboard` | Cookie tidak terkirim (beda domain / `Secure` / `SameSite`). Cek DevTools → Application → Cookies. |
| Env tidak terbaca | Variabel client harus diawali `NEXT_PUBLIC_`. Restart `npm run dev` setelah ubah `.env.local`. |

---

## Lisensi

Private — semua hak dilindungi. Hubungi pemilik repo (`bbal10/nubiacademy`) untuk izin penggunaan.
