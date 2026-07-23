# SITEMAP - Platform Digital Masjid

## Overview

Platform terdiri dari 3 aplikasi terpisah, masing-masing di-deploy ke subdomain berbeda:

| App | Domain | Target User | Rendering |
|-----|--------|-------------|-----------|
| Landing | `masjid.id` | Publik | SSG + ISR |
| Admin | `admin.masjid.id` | Pengurus, Pengajar | CSR (SPA) |
| Jamaah | `app.masjid.id` | Orang Tua, Donatur | CSR (PWA) |

---

## Landing App -- `masjid.id`

### Navbar

```
+------------------------------------------------------------------+
|  Logo    Tentang    Program    Keuangan    Artikel   [Daftar ->]  |
+------------------------------------------------------------------+
```

- **Logo**: Link ke `/` (home)
- **Tentang**: Link ke `/tentang`
- **Program**: Link ke `/program`
- **Keuangan**: Link ke `/keuangan`
- **Artikel**: Link ke `/artikel`
- **Daftar**: CTA button, link ke `/daftar` (yang redirect ke jamaah `/register`)

Mobile navbar: hamburger menu (slide-in dari kanan)

### Footer

```
+------------------------------------------------------------------+
|  Masjid [Nama]                                                    |
|  [Alamat lengkap masjid]                                          |
|                                                                    |
|  Tentang          Program           Informasi                      |
|  - Sejarah        - Ngaji            - Artikel                     |
|  - Visi Misi      - Belajar          - Keuangan                   |
|  - Pengurus         Kelompok                                       |
|                                                                    |
|  Legal                              Kontak                         |
|  - Syarat & Ketentuan               - WhatsApp: +62xxx             |
|  - Kebijakan Privasi                - Instagram: @masjidxxx        |
|  - Kebijakan Donasi                 - YouTube: @masjidxxx          |
|  - Kebijakan Cookie                                                |
|                                                                    |
|  (c) 2026 Masjid [Nama]. All rights reserved.                     |
+------------------------------------------------------------------+
```

### Page Map

| Route | Nama | Rendering | Deskripsi |
|-------|------|-----------|-----------|
| `/` | Home | SSG + ISR (1h) | Hero banner dengan foto masjid + tagline. Ringkasan section: tentang singkat, program unggulan (3-4 cards), statistik (jumlah santri, program), artikel terbaru (3 cards). CTA "Daftar Sekarang" yang prominent. |
| `/tentang` | Tentang | SSG | Sejarah singkat masjid, visi & misi, struktur kepengurusan (foto + nama + jabatan), info kontak (alamat, WA, maps embed) di bagian bawah. |
| `/program` | Program & Proyek | SSG + ISR (1h) | Single page dengan tabs: **Tab Program** menampilkan daftar program ngaji (Iqra, Al-Quran, Hafalan, Doa, Hadits) dan belajar kelompok sebagai cards. **Tab Proyek** menampilkan daftar proyek pembangunan dengan progress bar. |
| `/program/[slug]` | Detail Program | SSG + ISR (1h) | Detail satu program: deskripsi lengkap, jadwal, pengajar, syarat, cara mendaftar. Breadcrumb: Home > Program > [Nama]. `generateStaticParams()` untuk semua program aktif. |
| `/proyek/[slug]` | Detail Proyek | SSG + ISR (30m) | Detail proyek pembangunan: deskripsi, target dana vs terkumpul (progress bar), timeline milestones, gallery foto update, CTA donasi. Breadcrumb: Home > Program > [Nama Proyek]. |
| `/keuangan` | Keuangan | SSG + ISR (30m) | **Bagian atas**: Form/CTA donasi -- pilih kategori (infaq, sedekah, zakat, proyek), input nominal, metode pembayaran. **Bagian bawah**: Transparansi keuangan -- ringkasan pemasukan (SPP + donasi) vs pengeluaran, tabel riwayat transaksi (publik, tanpa data personal). |
| `/artikel` | Artikel | SSG + ISR (15m) | Blog list dengan pagination. Card per artikel: cover image, judul, excerpt, tanggal, tags. Search/filter by tag. |
| `/artikel/[slug]` | Detail Artikel | SSG + ISR (1h) | Full article: judul, cover image, konten (rich text), tanggal publish, author, tags. Related articles di bawah. Share buttons. Breadcrumb: Home > Artikel > [Judul]. |
| `/daftar` | Daftar | SSG | Halaman CTA pendaftaran santri baru. Menjelaskan benefit program, kemudian redirect/link ke jamaah app `/register`. Bukan form registrasi langsung. |
| `/legal/syarat-ketentuan` | Syarat & Ketentuan | SSG | Halaman legal: syarat penggunaan platform. Static content, hanya di-link dari footer. **Tidak muncul di navbar.** |
| `/legal/kebijakan-privasi` | Kebijakan Privasi | SSG | Halaman legal: kebijakan privasi data pengguna. Compliance GDPR-like. **Tidak muncul di navbar.** |
| `/legal/kebijakan-donasi` | Kebijakan Donasi | SSG | Halaman legal: ketentuan donasi, refund policy, penggunaan dana. **Tidak muncul di navbar.** |
| `/legal/kebijakan-cookie` | Kebijakan Cookie | SSG | Halaman legal: penggunaan cookie dan tracking. **Tidak muncul di navbar.** |
| `/sitemap.xml` | Sitemap | Dynamic | Auto-generated sitemap XML untuk semua public pages. Di-submit ke Google Search Console. |
| `/robots.txt` | Robots | Dynamic | Allow all crawlers, disallow `/api/` dan `/_next/`. Link ke sitemap. |

---

## Admin App -- `admin.masjid.id`

### Sidebar Navigation

```
+--------------------+
|  Masjid Digital    |
|  Admin Panel       |
+--------------------+
|  Dashboard         |
|  Santri            |
|  Kegiatan          |
|    - Program       |
|    - Kelas         |
|    - Jadwal        |
|    - Sesi          |
|  Keuangan          |
|    - SPP           |
|    - Donasi        |
|    - Pengeluaran   |
|    - Laporan       |
|  Proyek            |
|  Konten            |
|    - Pengumuman    |
|    - Galeri        |
|    - Landing       |
|    - Artikel       |
|  Settings          |
+--------------------+
|  Ustadz Ahmad      |
|  Pengurus          |
|  [Logout]          |
+--------------------+
```

### Top Bar

```
+----------------------------------------------------------+
|  =  Dashboard                              Notif(5)  [U] |
+----------------------------------------------------------+
```

### Page Map

| Route | Nama | Auth Required | Roles | Deskripsi |
|-------|------|:------------:|-------|-----------|
| `/` | Redirect | - | - | Redirect ke `/dashboard` |
| `/login` | Login | No | - | Form login: input WA + OTP + PIN. Redirect ke dashboard setelah berhasil. |
| `/dashboard` | Dashboard | Yes | All admin roles | Overview statistik: total santri aktif, kehadiran hari ini (%), sesi hari ini, keuangan bulan ini (pemasukan vs pengeluaran), chart kehadiran mingguan, notifikasi terbaru. |
| `/santri` | List Santri | Yes | superadmin, pengurus, pengajar | Tabel santri dengan search, filter (kelas, program, status aktif). Columns: nama, kelas, program, orang tua, status. Tombol "Tambah Santri". |
| `/santri/baru` | Tambah Santri | Yes | superadmin, pengurus | Form multi-step: data anak -> data orang tua (baru/existing) -> pilih program/kelas -> generate credentials -> kirim WA. |
| `/santri/[id]` | Detail Santri | Yes | superadmin, pengurus, pengajar | Tabs: **Profil** (data anak + orang tua), **Kehadiran** (calendar + statistik), **Progress Ngaji** (timeline), **Progress Belajar** (per mata pelajaran), **Keuangan** (tagihan SPP). Tombol edit, nonaktifkan. |
| `/kegiatan` | Overview Kegiatan | Yes | All admin roles | Overview: program aktif, kelas, jadwal hari ini, sesi terbaru. Quick action: mulai sesi. |
| `/kegiatan/program` | Program | Yes | superadmin, pengurus | CRUD program (ngaji: Iqra, Al-Quran, Hafalan, Doa, Hadits; belajar kelompok: per mata pelajaran). Toggle public/private untuk landing display. |
| `/kegiatan/kelas` | Kelas | Yes | superadmin, pengurus | CRUD kelas per program. Assign pengajar, daftar santri per kelas, max capacity. |
| `/kegiatan/jadwal` | Jadwal | Yes | superadmin, pengurus | Jadwal mingguan per kelas. Calendar view (week). Drag-drop untuk reschedule. |
| `/kegiatan/sesi` | Sesi | Yes | All admin roles | List sesi hari ini / minggu ini. Filter by kelas, pengajar, status. Tombol "Mulai Sesi" -> masuk ke attendance input. |
| `/kegiatan/sesi/[id]` | Detail Sesi | Yes | All admin roles | Attendance input grid + progress input per santri (ngaji atau belajar tergantung program). |
| `/keuangan` | Overview Keuangan | Yes | superadmin, pengurus | Summary: total pemasukan bulan ini (SPP + donasi), total pengeluaran, saldo. Chart bulanan. |
| `/keuangan/spp` | SPP / Bisyaroh | Yes | superadmin, pengurus | Manage fee types, generate invoices per bulan, track pembayaran. Tabel: santri, bulan, nominal, status, aksi (mark paid). Bulk generate invoices. |
| `/keuangan/donasi` | Donasi | Yes | superadmin, pengurus | List donasi masuk. Filter by kategori, status verifikasi. Verify donasi (dengan bukti transfer). |
| `/keuangan/pengeluaran` | Pengeluaran | Yes | superadmin, pengurus | Catat pengeluaran operasional. Kategori, deskripsi, nominal, tanggal, upload bukti. |
| `/keuangan/laporan` | Laporan Keuangan | Yes | superadmin, pengurus | Laporan bulanan/tahunan. Filter periode. Export PDF/Excel. Pemasukan vs pengeluaran, breakdown per kategori. |
| `/proyek` | List Proyek | Yes | superadmin, pengurus | CRUD proyek pembangunan. Card per proyek: nama, target dana, progress, status. |
| `/proyek/[id]` | Detail Proyek | Yes | superadmin, pengurus | Edit proyek, manage milestones (add/update/complete), post update (teks + upload foto). Preview tampilan publik. |
| `/konten` | Overview Konten | Yes | superadmin, pengurus | Quick access ke semua jenis konten. |
| `/konten/pengumuman` | Pengumuman | Yes | superadmin, pengurus | CRUD pengumuman. Target audience (role-based), pin pengumuman, set tanggal publish/expire. |
| `/konten/galeri` | Galeri | Yes | superadmin, pengurus | Upload & manage foto. Kategorisasi (kegiatan, fasilitas, acara). Toggle public/private. |
| `/konten/landing` | Edit Landing | Yes | superadmin, pengurus | Edit konten halaman landing (hero text, tentang, dll) via CMS-like interface. Edit pages table. |
| `/konten/artikel` | Artikel | Yes | superadmin, pengurus | CRUD artikel blog. Rich text editor, cover image upload, tags, SEO meta (title, description). Draft/publish/archive status. |
| `/settings` | Settings | Yes | superadmin | User management (CRUD users, assign roles), konfigurasi sistem (nama masjid, alamat, nomor WA, social media), manage fee types. |

---

## Jamaah App -- `app.masjid.id`

### Bottom Navigation

```
+------------------------------------------+
|  Home     Anak     Bayar   Masjid  Profil|
+------------------------------------------+
```

- **Home** (`/`): Default active tab
- **Anak** (`/anak`): List & detail anak
- **Bayar** (`/bayar`): SPP & donasi
- **Masjid** (`/masjid`): Info masjid
- **Profil** (`/profil`): User settings

Badge notification count muncul di icon Home atau Profil.

### Page Map

| Route | Nama | Auth Required | Roles | Deskripsi |
|-------|------|:------------:|-------|-----------|
| `/login` | Login | No | - | Form login: input WA -> OTP -> PIN. Minimal UI, fokus ke input. Tombol "Belum punya akun? Daftar". |
| `/register` | Registrasi | No | - | Multi-step onboarding: **Step 1** Input nama + no WA. **Step 2** Verifikasi OTP (6 digit, dikirim via WA). **Step 3** Set PIN 6 digit + konfirmasi. **Step 4** Sukses -> redirect ke `/anak` untuk tambah anak. |
| `/` | Home | Yes | orangtua, donatur | **Greeting**: "Assalamu'alaikum, [Nama]". **Stat cards** (2x2 grid): jumlah anak, kehadiran bulan ini (%), tagihan SPP pending, progress terakhir. **Kegiatan hari ini**: list jadwal anak hari ini (jam, program, kelas, lokasi). **Feed terbaru**: pengumuman, update progress, update proyek -- chronological feed. |
| `/anak` | List Anak | Yes | orangtua | List cards anak yang terdaftar. Per card: foto/avatar, nama, kelas saat ini, kehadiran bulan ini (mini bar). Tombol "Tambah Anak" (untuk self-register flow). |
| `/anak/[id]` | Detail Anak | Yes | orangtua | **Tabs**: Kehadiran, Ngaji, Belajar, Rapor. **Kehadiran tab**: Calendar view bulan ini (dot indicator per hari: hijau=hadir, kuning=izin, merah=alfa), statistik kehadiran, riwayat absensi. **Ngaji tab**: Progress terakhir per program (Iqra: jilid X hal Y, Hafalan: surah X status Y), timeline history. **Belajar tab**: Progress per mata pelajaran, nilai terbaru, grafik perkembangan. **Rapor tab**: List rapor per semester/periode, bisa di-view detail. Tombol floating "Ajukan Izin" -> ke form izin. |
| `/anak/[id]/izin` | Form Izin | Yes | orangtua | Form pengajuan izin: pilih tanggal (date picker), pilih alasan (sakit/izin), catatan opsional. Submit -> notifikasi ke pengajar. Bisa lihat status izin sebelumnya (pending/approved/rejected). |
| `/bayar` | Pembayaran | Yes | orangtua, donatur | **Section SPP** (orangtua): List tagihan SPP per anak, per bulan. Status (pending/paid/overdue). Tap untuk detail -> metode pembayaran (transfer manual, upload bukti). **Section Donasi** (semua): Form donasi: pilih kategori, input nominal, opsional pilih proyek. Riwayat donasi user. |
| `/masjid` | Info Masjid | Yes | orangtua, donatur | **Proyek terbaru**: Card per proyek aktif, progress bar, link ke detail. **Transparansi keuangan**: Ringkasan pemasukan/pengeluaran bulan ini (read-only view dari data publik). **Pengumuman**: List pengumuman aktif (yang target role-nya match). |
| `/notifikasi` | Notifikasi | Yes | orangtua, donatur | List notifikasi chronological. Jenis: attendance (anak tidak hadir), progress (update ngaji/belajar), payment (tagihan/reminder), announcement (pengumuman baru), project (update proyek). Swipe untuk mark as read. Unread indicator (bold + dot). |
| `/profil` | Profil | Yes | orangtua, donatur | **Info profil**: Nama, no WA, avatar. Tombol edit. **Kelola anak**: List anak terdaftar, link ke detail. **Ganti PIN**: Form ganti PIN (old PIN -> new PIN -> confirm). **Tentang app**: Versi, link ke legal pages. **Logout**: Konfirmasi sebelum logout. |

---

## Route Protection Summary

### Public Routes (No Auth)
```
Landing:  /* (semua halaman landing)
Admin:    /login
Jamaah:   /login, /register
```

### Protected Routes (Auth Required)
```
Admin:    /dashboard, /santri/*, /kegiatan/*, /keuangan/*, /proyek/*, /konten/*, /settings
Jamaah:   /, /anak/*, /bayar, /masjid, /notifikasi, /profil
```

### Role-Based Routes

```
superadmin only:
  Admin: /settings (full access)

superadmin + pengurus:
  Admin: /santri/baru, /keuangan/*, /proyek/*, /konten/*

superadmin + pengurus + pengajar:
  Admin: /dashboard, /santri (read), /kegiatan/*

orangtua:
  Jamaah: /, /anak/*, /bayar (SPP section), /masjid, /notifikasi, /profil

donatur:
  Jamaah: /, /bayar (donasi section), /masjid, /notifikasi, /profil
```

---

## Rendering Strategy Summary

```
SSG (Static Site Generation):
  Landing: /tentang, /daftar, /legal/*

SSG + ISR (Incremental Static Regeneration):
  Landing: /, /program, /program/[slug], /proyek/[slug],
           /keuangan, /artikel, /artikel/[slug]

CSR (Client-Side Rendering):
  Admin:   All pages (SPA behind auth)
  Jamaah:  All pages (PWA behind auth)

Dynamic:
  Landing: /sitemap.xml, /robots.txt
```

---

## URL Design Principles

1. **Indonesian language** -- URL pakai Bahasa Indonesia untuk SEO lokal
2. **Slug-based** -- Dynamic content pakai slug (human-readable)
3. **No trailing slashes** -- Konsisten tanpa trailing slash
4. **Lowercase** -- Semua lowercase
5. **Hyphenated** -- Pakai hyphen, bukan underscore
6. **Short** -- Sependek mungkin tapi tetap descriptive

```
Good:  /program/tahfidz-al-quran
Bad:   /program/detail/tahfidz_al_quran/
Good:  /artikel/tips-mengajarkan-anak-ngaji
Bad:   /artikel/detail?id=123
```
