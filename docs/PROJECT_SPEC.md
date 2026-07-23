# PROJECT SPEC - Platform Digital Masjid

## Ringkasan Proyek

Platform digital masjid (Masjid Digital Platform) adalah sistem terintegrasi untuk mengelola seluruh kegiatan operasional masjid, mulai dari manajemen santri, program ngaji, keuangan (SPP/donasi), hingga transparansi proyek pembangunan. Platform ini dibangun sebagai monorepo Next.js dengan Supabase sebagai backend tunggal.

Platform ini terdiri dari 3 aplikasi:

| App | Target User | Tipe |
|-----|------------|------|
| **Landing** | Publik (calon jamaah, donatur) | Website SSG+ISR, SEO-optimized |
| **Admin** | Pengurus & Pengajar masjid | Dashboard SPA |
| **Jamaah** | Orang tua & Donatur | PWA mobile-first |

---

## Goals & Objectives

### Primary Goals
1. **Digitalisasi manajemen santri** — Data anak, kehadiran, progress ngaji & belajar kelompok, rapor digital
2. **Transparansi keuangan** — SPP/bisyaroh, donasi, pengeluaran, semua tercatat dan bisa dilihat publik
3. **Kemudahan komunikasi** — Notifikasi real-time via WhatsApp & in-app, pengumuman, feed kegiatan
4. **Self-service orang tua** — Orang tua bisa daftar sendiri, cek progress anak, bayar SPP, izinkan anak
5. **Aksesibilitas publik** — Landing page SEO-optimized untuk menarik jamaah & donatur baru

### Secondary Goals
- Mengurangi beban administrasi pengurus masjid
- Menyediakan rapor digital per semester
- Tracking proyek pembangunan masjid secara transparan
- Membangun komunitas jamaah yang engaged

---

## Stakeholders & Roles

### Role Definitions

| Role | Deskripsi | Akses App |
|------|-----------|-----------|
| **superadmin** | Full access, manage users, config sistem | Admin |
| **pengurus** | Kelola keuangan, program, konten, proyek | Admin |
| **pengajar** | Input kehadiran, progress ngaji & belajar, manage sesi | Admin |
| **orangtua** | Lihat progress anak, bayar SPP, izin anak | Jamaah |
| **donatur** | Donasi, lihat transparansi, update proyek | Jamaah |

### Role Hierarchy
```
superadmin
  └── pengurus
        └── pengajar
orangtua (independent)
donatur (independent)
```

Satu user bisa punya multiple roles. Misal: seorang orangtua juga bisa jadi donatur. Pengajar juga bisa jadi orangtua.

---

## Feature Breakdown per App

### Landing App (Public Website)

| Fitur | Halaman | Deskripsi |
|-------|---------|-----------|
| Hero Section | `/` | Banner utama + ringkasan masjid + CTA daftar |
| Tentang Masjid | `/tentang` | Sejarah, visi misi, struktur pengurus, kontak |
| Program & Proyek | `/program` | Tab view: Program ngaji/belajar & Proyek pembangunan |
| Detail Program | `/program/[slug]` | Detail lengkap satu program |
| Detail Proyek | `/proyek/[slug]` | Detail proyek + milestone + progress |
| Keuangan | `/keuangan` | Form donasi (atas) + transparansi keuangan (bawah) |
| Artikel | `/artikel` | Blog list dengan pagination |
| Detail Artikel | `/artikel/[slug]` | Full article dengan structured data |
| Pendaftaran | `/daftar` | CTA redirect ke jamaah app `/register` |
| Legal Pages | `/legal/*` | Syarat ketentuan, privasi, donasi, cookie |

### Admin App (Dashboard)

| Fitur | Halaman | Deskripsi |
|-------|---------|-----------|
| Dashboard | `/dashboard` | Overview statistik: santri aktif, kehadiran hari ini, keuangan bulan ini |
| Manajemen Santri | `/santri` | CRUD anak + data orang tua + generate PIN login |
| Kegiatan | `/kegiatan` | Manage program, kelas, jadwal mingguan, sesi harian |
| Keuangan | `/keuangan` | SPP invoicing, pencatatan donasi, pengeluaran, laporan |
| Proyek | `/proyek` | CRUD proyek, milestone, update progress, upload foto |
| Konten | `/konten` | Pengumuman, galeri foto, edit landing page content, artikel blog |
| Settings | `/settings` | User management, role assignment, konfigurasi sistem |

### Jamaah App (PWA)

| Fitur | Halaman | Deskripsi |
|-------|---------|-----------|
| Home | `/` | Stats ringkasan + kegiatan hari ini + feed terbaru |
| List Anak | `/anak` | Daftar anak yang terdaftar |
| Detail Anak | `/anak/[id]` | Kehadiran, progress ngaji, progress belajar, rapor |
| Form Izin | `/anak/[id]/izin` | Form pengajuan izin tidak hadir |
| Pembayaran | `/bayar` | Bayar SPP + donasi |
| Info Masjid | `/masjid` | Update proyek, transparansi keuangan, pengumuman |
| Notifikasi | `/notifikasi` | List notifikasi (in-app) |
| Profil | `/profil` | Edit profil, kelola anak, ganti PIN |
| Registrasi | `/register` | Onboarding: nama + WA + OTP + PIN |

---

## User Flows

### 1. Flow Registrasi

#### Self-Register (Orang Tua)
```
Landing /daftar → Redirect ke Jamaah /register
  → Input nama lengkap
  → Input nomor WhatsApp
  → Kirim OTP via WhatsApp
  → Input 6-digit OTP
  → Set 6-digit PIN
  → Profil created (role: orangtua)
  → Redirect ke /anak → Tambah data anak
  → Admin approve/verify anak
```

#### Admin-Register (Admin mendaftarkan anak + ortu)
```
Admin /santri → Tambah Anak Baru
  → Input data anak (nama, tanggal lahir, dll)
  → Input data orang tua (nama, no WA)
  → Sistem generate credentials (PIN)
  → Kirim credentials via WhatsApp ke ortu
  → Ortu login pertama kali → verify OTP → set PIN baru
```

### 2. Flow Kehadiran (Attendance)

#### Pengajar: Session-Based Attendance
```
Admin /kegiatan → Pilih Kelas → Mulai Sesi
  → List santri muncul
  → Toggle hadir/tidak hadir per santri
  → Catatan opsional per santri
  → Submit → attendance records created
  → Notifikasi otomatis ke ortu yang anaknya tidak hadir
```

#### Orang Tua: Leave Request (Izin)
```
Jamaah /anak/[id]/izin
  → Pilih tanggal
  → Pilih alasan (sakit, izin, lainnya)
  → Catatan opsional
  → Submit → leave request created
  → Admin/pengajar bisa lihat di sesi
```

### 3. Flow Progress Al-Quran (Ngaji)

```
Pengajar buka sesi ngaji → Pilih santri
  → Input progress:
    - Program: Iqra / Al-Quran / Hafalan / Doa / Hadits
    - Untuk Iqra: jilid + halaman
    - Untuk Al-Quran: surah + ayat
    - Untuk Hafalan: surah target + status (belum/proses/hafal)
    - Untuk Doa/Hadits: nama doa/hadits + status
    - Nilai/grade opsional
    - Catatan pengajar
  → Save → progress record created
  → Ortu bisa lihat real-time di jamaah app
```

### 4. Flow Progress Belajar Kelompok (Study)

```
Pengajar buka sesi belajar kelompok → Pilih kelas
  → Input progress per santri:
    - Mata pelajaran (Matematika, IPA, Bahasa, dll)
    - Topik/materi hari ini
    - Nilai/assessment opsional
    - Catatan pengajar
  → Save → study_progress record created
  → Ortu bisa lihat di jamaah app
```

### 5. Flow Keuangan (SPP / Donasi)

#### SPP (Bisyaroh) - Admin Side
```
Admin /keuangan → SPP
  → Set fee types (bulanan, pendaftaran, dll)
  → Generate invoice per santri per bulan
  → Track pembayaran (manual/transfer)
  → Mark as paid
  → Otomatis notifikasi ortu
```

#### SPP - Orang Tua Side
```
Jamaah /bayar → Lihat tagihan SPP
  → Pilih tagihan → Bayar
  → (Future: payment gateway integration)
  → Untuk sekarang: transfer manual → upload bukti
  → Admin verify → mark paid
```

#### Donasi
```
Landing /keuangan atau Jamaah /bayar
  → Pilih kategori donasi (infaq, sedekah, zakat, proyek)
  → Input nominal
  → (Future: payment gateway)
  → Donasi tercatat → muncul di transparansi publik
```

### 6. Flow Proyek Pembangunan

```
Admin /proyek → Buat Proyek Baru
  → Nama, deskripsi, target dana, timeline
  → Tambah milestones
  → Post update berkala (teks + foto)
  → Progress auto-calculate dari milestones

Publik (Landing /proyek/[slug]):
  → Lihat detail proyek
  → Timeline milestones
  → Gallery foto update
  → Progress bar dana terkumpul
  → CTA donasi

Jamaah /masjid:
  → Feed update proyek terbaru
  → Detail proyek + milestone
```

---

## Tech Stack

### Core Stack

| Layer | Technology | Keterangan |
|-------|-----------|------------|
| **Monorepo** | Turborepo + pnpm | Shared packages, parallel builds |
| **Framework** | Next.js 14+ (App Router) | SSG/ISR untuk landing, SPA untuk admin/jamaah |
| **Language** | TypeScript | Strict mode, shared types |
| **Styling** | Tailwind CSS + shadcn/ui | Mobile-first, customizable |
| **Backend** | Supabase | Single project, shared across apps |
| **Database** | PostgreSQL (via Supabase) | RLS, triggers, functions |
| **Auth** | Supabase Auth (Phone) | WhatsApp OTP + PIN |
| **Realtime** | Supabase Realtime | Live updates untuk attendance, notifications |
| **Storage** | Supabase Storage | Foto profil, galeri, bukti transfer |
| **Edge Functions** | Supabase Edge Functions (Deno) | OTP, notifications, credential generation |
| **Deployment** | Vercel | Per-app deployment |
| **CI/CD** | GitHub Actions | Lint, test, build, deploy |

### Packages (Shared)

| Package | Isi |
|---------|-----|
| `@repo/ui` | shadcn/ui components + custom components |
| `@repo/db` | Supabase client, types, queries |
| `@repo/auth` | Auth utilities, hooks, middleware |
| `@repo/config` | Shared configs (Tailwind, ESLint, TypeScript) |
| `@repo/utils` | Helper functions, formatters, validators |

---

## Deployment Strategy

### Environment Setup
- **Production**: Main branch → auto-deploy ke Vercel
- **Staging**: `staging` branch → preview deployment
- **Development**: Feature branches → preview URLs

### Per-App Deployment (Vercel)
```
landing.masjid.id    → apps/landing (Vercel Project 1)
admin.masjid.id      → apps/admin   (Vercel Project 2)
app.masjid.id        → apps/jamaah  (Vercel Project 3)
```

### Supabase
- Single Supabase project untuk semua environments
- Branching database untuk staging (Supabase Branching)
- Edge Functions di-deploy via `supabase functions deploy`

---

## Phase Plan

### Phase 1: Foundation (Week 1-3)
- [x] Setup monorepo (Turborepo + pnpm)
- [x] Setup Supabase project
- [x] Database schema + migrations
- [x] Auth flow (phone + OTP + PIN)
- [x] Shared packages (@repo/ui, @repo/db, @repo/auth)
- [x] Landing page basic (home, tentang)
- [x] Admin login + dashboard skeleton

### Phase 2: Core Features (Week 4-7)
- [ ] Admin: CRUD santri + orang tua
- [ ] Admin: Manajemen program & kelas
- [ ] Admin: Session & attendance input
- [ ] Jamaah: Registration flow
- [ ] Jamaah: Home + list anak
- [ ] Jamaah: Detail anak (attendance view)
- [ ] Landing: Program & tentang pages

### Phase 3: Academic Features (Week 8-10)
- [ ] Admin: Input progress ngaji (Iqra, Al-Quran, Hafalan, Doa, Hadits)
- [ ] Admin: Input progress belajar kelompok
- [ ] Jamaah: View progress ngaji & belajar
- [ ] Admin: Generate rapor digital
- [ ] Jamaah: View rapor
- [ ] Jamaah: Form izin anak
- [ ] Real-time notifications (attendance, progress)

### Phase 4: Finance & Content (Week 11-14)
- [ ] Admin: Fee types & SPP invoicing
- [ ] Admin: Pencatatan donasi & pengeluaran
- [ ] Admin: Laporan keuangan
- [ ] Jamaah: View tagihan & bayar (manual)
- [ ] Landing: Keuangan page (donasi + transparansi)
- [ ] Admin: Proyek management + milestones
- [ ] Landing: Detail proyek
- [ ] Admin: Artikel & konten management
- [ ] Landing: Blog/artikel

### Phase 5: Polish & Scale (Week 15-18)
- [ ] PWA optimization (offline, install prompt)
- [ ] Push notifications (web push + WhatsApp)
- [ ] Payment gateway integration (future)
- [ ] Performance optimization (Core Web Vitals)
- [ ] SEO optimization (structured data, sitemap)
- [ ] Analytics integration
- [ ] Legal pages
- [ ] User feedback & iteration
- [ ] Documentation & handoff

---

## Non-Functional Requirements

### Performance
- Landing page: LCP < 2.5s, FID < 100ms, CLS < 0.1
- Admin/Jamaah: Initial load < 3s on 3G
- API response time < 500ms (p95)

### Security
- Row Level Security (RLS) di semua tabel
- PIN hashed (bcrypt) di database
- OTP expire dalam 5 menit
- Rate limiting pada auth endpoints
- CORS configured per app domain

### Accessibility
- WCAG 2.1 Level AA compliance
- Keyboard navigable
- Screen reader friendly
- Color contrast minimum 4.5:1

### Scalability
- Target: 500+ santri, 200+ orang tua concurrent
- Database indexing pada query-heavy columns
- ISR revalidation untuk landing pages
- CDN untuk static assets

---

## Conventions & Standards

### Code Style
- ESLint + Prettier (shared config)
- Conventional commits
- TypeScript strict mode
- Component naming: PascalCase
- File naming: kebab-case
- Database columns: snake_case

### Git Workflow
- Main branch: production
- Feature branches: `feat/feature-name`
- Bug fixes: `fix/bug-description`
- PR required, minimum 1 review
- Squash merge ke main

### Documentation
- JSDoc untuk public functions
- README per package
- Storybook untuk UI components (future)
