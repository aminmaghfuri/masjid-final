# UI/UX SPEC - Platform Digital Masjid

## Design Principles

### 1. Mobile-First, Native Feel
- Target utama: orang tua santri yang akses via HP
- UI harus terasa seperti native app, bukan website
- Bottom navigation untuk jamaah app
- Gesture-friendly (swipe, pull-to-refresh)
- Touch targets minimal 44x44px

### 2. Compact & High Density
- Informasi padat tapi tetap readable
- Stat cards dengan angka besar + label kecil
- Efficient use of space -- no unnecessary whitespace
- Cards sebagai primary container pattern
- Horizontal scroll untuk overflow content

### 3. Islamic Aesthetic, Modern Execution
- Clean, minimalis, tidak berlebihan ornamen
- Typography yang proper untuk teks Arab (Amiri font)
- Warna hijau (emerald/teal) sebagai primary -- identik dengan Islam
- Subtle patterns (geometric Islamic patterns) sebagai accent, bukan dominan

### 4. Accessibility & Inclusivity
- Color contrast WCAG AA (min 4.5:1)
- Font size minimum 14px untuk body text
- Clear visual hierarchy
- Loading states & skeleton screens
- Error states yang helpful

---

## Visual System

### Color Palette

#### Primary Colors (Emerald)
```css
--emerald-50:  #ecfdf5;
--emerald-100: #d1fae5;
--emerald-200: #a7f3d0;
--emerald-300: #6ee7b7;
--emerald-400: #34d399;
--emerald-500: #10b981;  /* Primary */
--emerald-600: #059669;  /* Primary Hover */
--emerald-700: #047857;
--emerald-800: #065f46;
--emerald-900: #064e3b;
--emerald-950: #022c22;
```

#### Secondary Colors (Teal)
```css
--teal-50:  #f0fdfa;
--teal-100: #ccfbf1;
--teal-200: #99f6e4;
--teal-300: #5eead4;
--teal-400: #2dd4bf;
--teal-500: #14b8a6;  /* Secondary */
--teal-600: #0d9488;  /* Secondary Hover */
--teal-700: #0f766e;
--teal-800: #115e59;
--teal-900: #134e4a;
--teal-950: #042f2e;
```

#### Semantic Colors
```css
--success:  #22c55e;  /* green-500 */
--warning:  #f59e0b;  /* amber-500 */
--error:    #ef4444;  /* red-500 */
--info:     #3b82f6;  /* blue-500 */
```

#### Neutral Colors
```css
--gray-50:  #f9fafb;
--gray-100: #f3f4f6;
--gray-200: #e5e7eb;
--gray-300: #d1d5db;
--gray-400: #9ca3af;
--gray-500: #6b7280;
--gray-600: #4b5563;
--gray-700: #374151;
--gray-800: #1f2937;
--gray-900: #111827;
--gray-950: #030712;
```

#### Dark Mode
```css
/* Background */
--bg-primary:     #030712;   /* gray-950 */
--bg-secondary:   #111827;   /* gray-900 */
--bg-card:        #1f2937;   /* gray-800 */
--bg-elevated:    #374151;   /* gray-700 */

/* Text */
--text-primary:   #f9fafb;   /* gray-50 */
--text-secondary: #d1d5db;   /* gray-300 */
--text-muted:     #9ca3af;   /* gray-400 */

/* Primary stays the same */
--primary-dark:   #34d399;   /* emerald-400, brighter for dark bg */
```

### Typography

#### Font Stack
```css
/* Primary font - UI text */
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

/* Arabic font - Quran text, doa */
--font-arabic: 'Amiri', 'Traditional Arabic', serif;

/* Monospace - for codes, PINs */
--font-mono: 'JetBrains Mono', 'Fira Code', monospace;
```

#### Type Scale (8px base unit)
```css
--text-xs:    0.75rem;   /* 12px */
--text-sm:    0.875rem;  /* 14px */
--text-base:  1rem;      /* 16px - body */
--text-lg:    1.125rem;  /* 18px */
--text-xl:    1.25rem;   /* 20px */
--text-2xl:   1.5rem;    /* 24px */
--text-3xl:   1.875rem;  /* 30px */
--text-4xl:   2.25rem;   /* 36px - hero */

/* Line heights */
--leading-tight:  1.25;
--leading-normal: 1.5;
--leading-relaxed: 1.75;

/* Font weights */
--font-normal:   400;
--font-medium:   500;
--font-semibold: 600;
--font-bold:     700;
```

### Spacing (8px Grid System)

```css
--space-0:  0;
--space-1:  0.25rem;  /* 4px */
--space-2:  0.5rem;   /* 8px */
--space-3:  0.75rem;  /* 12px */
--space-4:  1rem;     /* 16px */
--space-5:  1.25rem;  /* 20px */
--space-6:  1.5rem;   /* 24px */
--space-8:  2rem;     /* 32px */
--space-10: 2.5rem;   /* 40px */
--space-12: 3rem;     /* 48px */
--space-16: 4rem;     /* 64px */
--space-20: 5rem;     /* 80px */
```

### Border Radius

```css
--radius-sm:   0.5rem;   /* 8px - small elements */
--radius-md:   0.75rem;  /* 12px - cards, containers */
--radius-lg:   1rem;     /* 16px - buttons, large cards */
--radius-xl:   1.5rem;   /* 24px - bottom sheets */
--radius-full: 9999px;   /* pill, avatar */
```

### Shadow & Elevation

```css
/* Light mode */
--shadow-sm:  0 1px 2px rgba(0, 0, 0, 0.05);
--shadow-md:  0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
--shadow-lg:  0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
--shadow-xl:  0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);

/* Glassmorphism effect */
--glass-bg: rgba(255, 255, 255, 0.8);
--glass-border: rgba(255, 255, 255, 0.2);
--glass-blur: blur(12px);
```

### Glassmorphism

Dipakai secara selektif untuk elemen yang perlu standout:

```css
.glass-card {
  background: rgba(255, 255, 255, 0.75);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.08);
}

/* Dark mode */
.dark .glass-card {
  background: rgba(31, 41, 55, 0.75);
  border: 1px solid rgba(255, 255, 255, 0.08);
}
```

Penggunaan glassmorphism:
- Stat cards di jamaah home
- Navigation bar (saat scroll)
- Modal/dialog overlay
- Bottom sheet header

---

## Component Library

### Bottom Navigation (Jamaah App)

```
+------------------------------------------+
|                                          |
|            [Page Content]                |
|                                          |
+------------------------------------------+
|  Home     Anak     Bayar   Masjid  Profil|
|  ----                                     |
+------------------------------------------+
```

```typescript
// packages/ui/src/components/bottom-nav.tsx

interface BottomNavProps {
  items: {
    label: string
    href: string
    icon: React.ReactNode
    badge?: number    // notification count
  }[]
  activeHref: string
}

export function BottomNav({ items, activeHref }: BottomNavProps) {
  return (
    <nav className="fixed bottom-0 left-0 right-0 z-50
                    bg-white/80 backdrop-blur-xl border-t border-gray-200
                    dark:bg-gray-900/80 dark:border-gray-800
                    pb-safe">
      <div className="flex justify-around items-center h-16 max-w-lg mx-auto">
        {items.map((item) => (
          <Link
            key={item.href}
            href={item.href}
            className={cn(
              "flex flex-col items-center gap-0.5 py-1 px-3 relative",
              "transition-colors duration-200",
              activeHref === item.href
                ? "text-emerald-600"
                : "text-gray-500"
            )}
          >
            <div className="relative">
              {item.icon}
              {item.badge && (
                <span className="absolute -top-1 -right-2 bg-red-500 text-white
                               text-[10px] font-bold rounded-full w-4 h-4
                               flex items-center justify-center">
                  {item.badge}
                </span>
              )}
            </div>
            <span className="text-[10px] font-medium">{item.label}</span>
            {activeHref === item.href && (
              <div className="absolute -top-px left-1/2 -translate-x-1/2
                            w-8 h-0.5 bg-emerald-600 rounded-full" />
            )}
          </Link>
        ))}
      </div>
    </nav>
  )
}
```

### Stat Card

```
+------------------+  +------------------+
|  Kehadiran       |  |  Progress        |
|                  |  |                  |
|     87%          |  |  Surah Al-Mulk   |
|  Bulan ini       |  |  Hari ini        |
+------------------+  +------------------+
```

```typescript
// packages/ui/src/components/stat-card.tsx

interface StatCardProps {
  title: string
  value: string | number
  subtitle?: string
  icon?: React.ReactNode
  trend?: {
    value: number
    isPositive: boolean
  }
  variant?: 'default' | 'emerald' | 'teal' | 'amber' | 'rose'
  className?: string
}

export function StatCard({
  title,
  value,
  subtitle,
  icon,
  trend,
  variant = 'default',
  className,
}: StatCardProps) {
  const variantStyles = {
    default: 'bg-white dark:bg-gray-800',
    emerald: 'bg-emerald-50 dark:bg-emerald-950/30',
    teal: 'bg-teal-50 dark:bg-teal-950/30',
    amber: 'bg-amber-50 dark:bg-amber-950/30',
    rose: 'bg-rose-50 dark:bg-rose-950/30',
  }

  return (
    <div className={cn(
      "rounded-xl p-4 border border-gray-100 dark:border-gray-700",
      variantStyles[variant],
      className
    )}>
      <div className="flex items-center justify-between mb-2">
        <span className="text-sm text-gray-500 dark:text-gray-400 font-medium">
          {title}
        </span>
        {icon && <span className="text-gray-400">{icon}</span>}
      </div>
      <div className="text-2xl font-bold text-gray-900 dark:text-white">
        {value}
      </div>
      {(subtitle || trend) && (
        <div className="flex items-center gap-2 mt-1">
          {trend && (
            <span className={cn(
              "text-xs font-medium",
              trend.isPositive ? "text-emerald-600" : "text-rose-600"
            )}>
              {trend.isPositive ? '+' : ''}{trend.value}%
            </span>
          )}
          {subtitle && (
            <span className="text-xs text-gray-500">{subtitle}</span>
          )}
        </div>
      )}
    </div>
  )
}
```

### Attendance Toggle

```
+----------------------------------------+
|  Ahmad Fauzi                   [H][I][S][A]|
|  Kelas Iqra Jilid 2                    |
+----------------------------------------+
|  Siti Fatimah                  [H][I][S][A]|
|  Kelas Iqra Jilid 3                    |
+----------------------------------------+
|  Muhammad Rizki                [H][I][S][A]|
|  Kelas Al-Quran            (ada izin)  |
+----------------------------------------+
```

H = Hadir (green), I = Izin (amber), S = Sakit (blue), A = Alfa (gray)

```typescript
// packages/ui/src/components/attendance-toggle.tsx

interface AttendanceToggleProps {
  childName: string
  childClass?: string
  status: 'hadir' | 'izin' | 'sakit' | 'alfa'
  hasLeaveRequest?: boolean
  onStatusChange: (status: AttendanceStatus) => void
  disabled?: boolean
}

const statusConfig = {
  hadir: { label: 'Hadir', color: 'bg-emerald-500', icon: 'CheckCircle' },
  izin:  { label: 'Izin',  color: 'bg-amber-500',   icon: 'Clock' },
  sakit: { label: 'Sakit', color: 'bg-blue-500',     icon: 'Heart' },
  alfa:  { label: 'Alfa',  color: 'bg-gray-300',     icon: 'XCircle' },
}

export function AttendanceToggle({
  childName,
  childClass,
  status,
  hasLeaveRequest,
  onStatusChange,
  disabled,
}: AttendanceToggleProps) {
  return (
    <div className="flex items-center justify-between p-3 rounded-xl
                    bg-white dark:bg-gray-800 border border-gray-100 dark:border-gray-700">
      <div className="flex-1 min-w-0">
        <p className="font-medium text-gray-900 dark:text-white truncate">
          {childName}
        </p>
        {childClass && (
          <p className="text-xs text-gray-500 dark:text-gray-400 mt-0.5">
            {childClass}
          </p>
        )}
        {hasLeaveRequest && (
          <Badge variant="outline" className="mt-1 text-[10px]">
            Ada izin dari ortu
          </Badge>
        )}
      </div>

      <div className="flex gap-1 ml-3">
        {Object.entries(statusConfig).map(([key, config]) => {
          const isActive = status === key
          return (
            <button
              key={key}
              onClick={() => onStatusChange(key as AttendanceStatus)}
              disabled={disabled}
              className={cn(
                "w-9 h-9 rounded-lg flex items-center justify-center transition-all",
                isActive
                  ? `${config.color} text-white shadow-sm`
                  : "bg-gray-50 text-gray-400 hover:bg-gray-100 dark:bg-gray-700"
              )}
              title={config.label}
            >
              {config.label[0]}
            </button>
          )
        })}
      </div>
    </div>
  )
}
```

### Quran Input

```
+------------------------------------------+
|  Progress Ngaji - Ahmad Fauzi            |
|                                          |
|  Program: [v Iqra          ]             |
|                                          |
|  +---- Iqra Mode ----------+            |
|  | Jilid : [v 3         ]  |            |
|  | Halaman: [  15       ]  |            |
|  +--------------------------+            |
|                                          |
|  Nilai:    [v Jayyid Jiddan]             |
|  Catatan:  [_______________]             |
|                                          |
|           [Simpan]                       |
+------------------------------------------+
```

```typescript
// packages/ui/src/components/quran-input.tsx

interface QuranInputProps {
  childId: string
  childName: string
  onSubmit: (data: QuranProgressInput) => void
  isLoading?: boolean
}

interface QuranProgressInput {
  program_type: QuranProgram
  // Iqra
  iqra_volume?: number
  iqra_page?: number
  // Al-Quran
  surah_number?: number
  ayat_from?: number
  ayat_to?: number
  // Hafalan
  hafalan_surah?: number
  hafalan_status?: HafalanStatus
  // Doa/Hadits
  item_name?: string
  item_status?: HafalanStatus
  // Common
  grade?: string
  notes?: string
}

export function QuranInput({ childId, childName, onSubmit, isLoading }: QuranInputProps) {
  const [programType, setProgramType] = useState<QuranProgram>('iqra')

  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-base">Progress Ngaji - {childName}</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        {/* Program Type Selector */}
        <Select value={programType} onValueChange={setProgramType}>
          <SelectTrigger><SelectValue placeholder="Pilih Program" /></SelectTrigger>
          <SelectContent>
            <SelectItem value="iqra">Iqra</SelectItem>
            <SelectItem value="al-quran">Al-Quran</SelectItem>
            <SelectItem value="hafalan">Hafalan</SelectItem>
            <SelectItem value="doa">Doa</SelectItem>
            <SelectItem value="hadits">Hadits</SelectItem>
          </SelectContent>
        </Select>

        {/* Dynamic fields based on program type */}
        {programType === 'iqra' && <IqraFields />}
        {programType === 'al-quran' && <AlQuranFields />}
        {programType === 'hafalan' && <HafalanFields />}
        {(programType === 'doa' || programType === 'hadits') && <DoaHaditsFields />}

        {/* Grade */}
        <Select>
          <SelectTrigger><SelectValue placeholder="Nilai" /></SelectTrigger>
          <SelectContent>
            <SelectItem value="mumtaz">Mumtaz (Istimewa)</SelectItem>
            <SelectItem value="jayyid_jiddan">Jayyid Jiddan (Sangat Baik)</SelectItem>
            <SelectItem value="jayyid">Jayyid (Baik)</SelectItem>
            <SelectItem value="maqbul">Maqbul (Cukup)</SelectItem>
            <SelectItem value="rasib">Rasib (Kurang)</SelectItem>
          </SelectContent>
        </Select>

        {/* Notes */}
        <Textarea placeholder="Catatan pengajar..." />

        <Button onClick={handleSubmit} disabled={isLoading} className="w-full">
          {isLoading ? 'Menyimpan...' : 'Simpan Progress'}
        </Button>
      </CardContent>
    </Card>
  )
}
```

---

## Wireframes

### 1. Jamaah Home Screen

```
+-------------------------------------+
|  Masjid [Nama]             Notif(3) |
+-------------------------------------+
|                                     |
|  Assalamu'alaikum, Bu Fatimah       |
|                                     |
|  +-----------+  +-----------+       |
|  | 2         |  | 87%       |       |
|  | Anak      |  | Kehadiran |       |
|  | Terdaftar |  | Bulan ini |       |
|  +-----------+  +-----------+       |
|  +-----------+  +-----------+       |
|  | Rp150K    |  | Juz 2     |       |
|  | Tagihan   |  | Progress  |       |
|  | SPP       |  | Hafalan   |       |
|  +-----------+  +-----------+       |
|                                     |
|  Kegiatan Hari Ini                  |
|  +-----------------------------+    |
|  | 15:30 - Ngaji Iqra          |    |
|  |    Ahmad - Kelas Jilid 3    |    |
|  |    Musholla Lantai 2        |    |
|  +-----------------------------+    |
|  | 16:30 - Belajar Kelompok    |    |
|  |    Siti - Kelas Matematika  |    |
|  |    Ruang Belajar 1          |    |
|  +-----------------------------+    |
|                                     |
|  Feed Terbaru                       |
|  +-----------------------------+    |
|  | Pengumuman                   |    |
|  | Jadwal libur Maulid Nabi    |    |
|  | 2 jam lalu                  |    |
|  +-----------------------------+    |
|  | Progress Ngaji               |    |
|  | Ahmad naik ke Iqra Jilid 4  |    |
|  | Kemarin                     |    |
|  +-----------------------------+    |
|                                     |
+-------------------------------------+
| Home    Anak   Bayar  Masjid  Profil|
+-------------------------------------+
```

### 2. Pengajar Session Screen (Admin App)

```
+-------------------------------------+
|  <- Sesi Ngaji                      |
|  Kelas Iqra Jilid 3                 |
|  Kamis, 20 Juli 2026 - 15:30-16:30  |
+-------------------------------------+
|                                     |
|  Daftar Hadir (12 santri)   [v All] |
|                                     |
|  +-----------------------------+    |
|  | Ahmad Fauzi      [H][I][S][A]|   |
|  | Iqra Jilid 3                |    |
|  +-----------------------------+    |
|  | Siti Aisyah      [H][I][S][A]|   |
|  | Iqra Jilid 3                |    |
|  +-----------------------------+    |
|  | Muh. Rizki       [H][I][S][A]|   |
|  | Iqra Jilid 3   (izin ortu) |    |
|  +-----------------------------+    |
|  | ... (9 santri lagi)         |    |
|  +-----------------------------+    |
|                                     |
|  +-----------------------------+    |
|  | Catatan Sesi                |    |
|  | [_________________________] |    |
|  +-----------------------------+    |
|                                     |
|  [      Simpan Kehadiran      ]     |
|                                     |
|  ---                                |
|                                     |
|  Input Progress Ngaji               |
|                                     |
|  [Pilih Santri v]                   |
|                                     |
|  +- Ahmad Fauzi ---------------+    |
|  | Program: [Iqra v]           |    |
|  | Jilid:   [3 v]  Hal: [15]  |    |
|  | Nilai:   [Jayyid Jiddan v]  |    |
|  | Catatan: [_______________]  |    |
|  |          [Simpan Progress]  |    |
|  +-----------------------------+    |
|                                     |
+-------------------------------------+
```

### 3. Orang Tua Form Izin (Jamaah App)

```
+-------------------------------------+
|  <- Pengajuan Izin                  |
|  Ahmad Fauzi                        |
+-------------------------------------+
|                                     |
|  Tanggal Izin                       |
|  +-----------------------------+    |
|  |  Kamis, 20 Juli 2026        |    |
|  +-----------------------------+    |
|                                     |
|  Alasan                             |
|  +-----------------------------+    |
|  |  ( ) Sakit                   |    |
|  |  (*) Izin                    |    |
|  +-----------------------------+    |
|                                     |
|  Keterangan (opsional)              |
|  +-----------------------------+    |
|  |  Kontrol ke dokter gigi     |    |
|  |                              |    |
|  +-----------------------------+    |
|                                     |
|  +-----------------------------+    |
|  |  Info: Izin akan dikirim ke  |    |
|  |  pengajar kelas terkait.     |    |
|  |  Status bisa dilihat di      |    |
|  |  halaman detail anak.        |    |
|  +-----------------------------+    |
|                                     |
|  [       Ajukan Izin        ]       |
|                                     |
+-------------------------------------+
| Home    Anak   Bayar  Masjid  Profil|
+-------------------------------------+
```

### 4. Pengajar Belajar Kelompok Session (Admin App)

```
+-------------------------------------+
|  <- Sesi Belajar Kelompok           |
|  Kelas Matematika SD                |
|  Kamis, 20 Juli 2026 - 16:30-17:30  |
+-------------------------------------+
|                                     |
|  Materi Hari Ini                    |
|  +-----------------------------+    |
|  | Mapel: [Matematika v]       |    |
|  | Topik: [Perkalian 2 digit ] |    |
|  +-----------------------------+    |
|                                     |
|  Daftar Hadir (8 santri)   [v All]  |
|  +-----------------------------+    |
|  | Ahmad Fauzi      [H][I][S][A]|   |
|  +-----------------------------+    |
|  | Siti Aisyah      [H][I][S][A]|   |
|  +-----------------------------+    |
|  | ...                         |    |
|  +-----------------------------+    |
|                                     |
|  [      Simpan Kehadiran      ]     |
|                                     |
|  ---                                |
|                                     |
|  Input Nilai                        |
|                                     |
|  +-----------------------------+    |
|  | Ahmad Fauzi                 |    |
|  | Nilai: [85    ] / 100       |    |
|  | Grade: [A v]                |    |
|  | Catatan: [Paham konsep ___] |    |
|  +-----------------------------+    |
|  | Siti Aisyah                 |    |
|  | Nilai: [72    ] / 100       |    |
|  | Grade: [B v]                |    |
|  | Catatan: [Perlu latihan __] |    |
|  +-----------------------------+    |
|                                     |
|  [     Simpan Semua Nilai     ]     |
|                                     |
+-------------------------------------+
```

### 5. Orang Tua Progress View (Jamaah App)

```
+-------------------------------------+
|  <- Ahmad Fauzi                     |
|  Kelas 3 SD                         |
+-------------------------------------+
|                                     |
|  [Kehadiran] [Ngaji] [Belajar] [Rapor]
|   ---------                         |
|                                     |
|  Kehadiran Bulan Juli 2026          |
|                                     |
|  Sen  Sel  Rab  Kam  Jum  Sab  Ahd |
|  ---  ---  ---  ---  ---  ---  ---  |
|            1    2    3    4    5     |
|            v    v    v         v     |
|  6    7    8    9    10   11   12    |
|       v    v    !    v         v     |
|  13   14   15   16   17   18   19   |
|       v    v    v    v         v     |
|  20   21   22   23                   |
|       x    -    -                    |
|                                     |
|  v = hadir, ! = izin, x = alfa      |
|                                     |
|  Ringkasan                          |
|  +-------+ +-------+ +-------+     |
|  | 14    | | 1     | | 1     |     |
|  | Hadir | | Izin  | | Alfa  |     |
|  +-------+ +-------+ +-------+     |
|                                     |
|  87.5% Kehadiran                    |
|  [================---]              |
|                                     |
|  ---                                |
|                                     |
|  Riwayat Kehadiran                  |
|  +-----------------------------+    |
|  | 21 Jul - Alfa               |    |
|  | Ngaji Iqra - Tanpa ket.    |    |
|  +-----------------------------+    |
|  | 9 Jul - Izin                |    |
|  | Ngaji Iqra - Kontrol dokter |    |
|  +-----------------------------+    |
|                                     |
+-------------------------------------+
| Home    Anak   Bayar  Masjid  Profil|
+-------------------------------------+
```

---

## Responsive Breakpoints

```css
/* Mobile first - default styles are for mobile */

/* Small (default) - 0-639px */
/* No @media needed, this is the base */

/* Medium - 640px+ (tablets) */
@media (min-width: 640px) { /* sm: */ }

/* Large - 768px+ (tablets landscape) */
@media (min-width: 768px) { /* md: */ }

/* Desktop - 1024px+ */
@media (min-width: 1024px) { /* lg: */ }

/* Wide desktop - 1280px+ */
@media (min-width: 1280px) { /* xl: */ }
```

### Layout Patterns per Breakpoint

| Breakpoint | Landing | Admin | Jamaah |
|------------|---------|-------|--------|
| Mobile (< 640px) | Single column, hamburger menu | N/A (not optimized) | Bottom nav, single column |
| Tablet (640-1023px) | 2-column grid | Collapsed sidebar | Bottom nav, 2-column cards |
| Desktop (1024px+) | Full layout, hero centered | Full sidebar + content | N/A (mobile PWA) |

### Container Max Widths

```css
/* Landing */
.container-landing {
  max-width: 1280px;  /* xl */
  margin: 0 auto;
  padding: 0 16px;
}

/* Admin */
.container-admin {
  max-width: 100%;    /* full width with sidebar */
  padding: 24px;
}

/* Jamaah */
.container-jamaah {
  max-width: 480px;   /* phone-optimized */
  margin: 0 auto;
  padding: 0 16px;
  padding-bottom: 80px; /* bottom nav space */
}
```

---

## Animation & Transitions

### Micro-interactions

```css
/* Button press effect */
.btn-press:active {
  transform: scale(0.97);
  transition: transform 100ms ease;
}

/* Card hover (desktop) */
.card-hover:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
  transition: all 200ms ease;
}

/* Toggle switch */
.toggle {
  transition: background-color 200ms ease;
}

/* Page transition */
.page-enter {
  opacity: 0;
  transform: translateY(8px);
}
.page-enter-active {
  opacity: 1;
  transform: translateY(0);
  transition: all 300ms ease-out;
}
```

### Loading States
- Skeleton screens (not spinners) untuk initial data load
- Inline spinners untuk button actions
- Pull-to-refresh untuk lists (jamaah app)
- Optimistic UI updates untuk toggles (attendance)

### Haptic Feedback (Mobile)
```typescript
// Attendance toggle success
if ('vibrate' in navigator) {
  navigator.vibrate(50)  // Short buzz
}
```

---

## Dark Mode

### Implementation

```typescript
// packages/ui/src/lib/theme.ts

export type Theme = 'light' | 'dark' | 'system'

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  return (
    <NextThemesProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      {children}
    </NextThemesProvider>
  )
}
```

### Dark Mode Color Mappings

| Element | Light | Dark |
|---------|-------|------|
| Background | white | gray-950 |
| Card | white | gray-800 |
| Border | gray-200 | gray-700 |
| Text primary | gray-900 | gray-50 |
| Text secondary | gray-500 | gray-400 |
| Primary | emerald-600 | emerald-400 |
| Stat card bg | emerald-50 | emerald-950/30 |

### PWA Configuration

```typescript
// apps/jamaah/app/manifest.ts
export default function manifest() {
  return {
    name: 'Masjid Digital',
    short_name: 'Masjid',
    theme_color: '#059669',        // emerald-600
    background_color: '#ffffff',
    display: 'standalone',
    orientation: 'portrait',
    start_url: '/',
    icons: [
      { src: '/icon-192.png', sizes: '192x192', type: 'image/png' },
      { src: '/icon-512.png', sizes: '512x512', type: 'image/png' },
    ],
  }
}
```
