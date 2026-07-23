# ARCHITECTURE - Platform Digital Masjid

## Overview

Arsitektur platform ini menggunakan pola monorepo dengan Turborepo sebagai build orchestrator dan pnpm workspaces untuk dependency management. Tiga aplikasi Next.js berbagi satu Supabase project sebagai backend tunggal, dengan shared packages untuk code reusability.

---

## Monorepo Structure

```
masjid-digital/
├── apps/
│   ├── landing/                    # Public website (SSG+ISR)
│   │   ├── app/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx            # / (home)
│   │   │   ├── tentang/
│   │   │   │   └── page.tsx        # /tentang
│   │   │   ├── program/
│   │   │   │   ├── page.tsx        # /program (tabs: program | proyek)
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx    # /program/[slug]
│   │   │   ├── proyek/
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx    # /proyek/[slug]
│   │   │   ├── keuangan/
│   │   │   │   └── page.tsx        # /keuangan
│   │   │   ├── artikel/
│   │   │   │   ├── page.tsx        # /artikel
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx    # /artikel/[slug]
│   │   │   ├── daftar/
│   │   │   │   └── page.tsx        # /daftar → redirect
│   │   │   ├── legal/
│   │   │   │   ├── syarat-ketentuan/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── kebijakan-privasi/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── kebijakan-donasi/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── kebijakan-cookie/
│   │   │   │       └── page.tsx
│   │   │   ├── sitemap.ts          # Dynamic sitemap
│   │   │   └── robots.ts           # Dynamic robots.txt
│   │   ├── components/
│   │   │   ├── navbar.tsx
│   │   │   ├── footer.tsx
│   │   │   ├── hero.tsx
│   │   │   ├── program-card.tsx
│   │   │   ├── donation-form.tsx
│   │   │   ├── transparency-table.tsx
│   │   │   └── article-card.tsx
│   │   ├── lib/
│   │   │   ├── metadata.ts
│   │   │   └── structured-data.ts
│   │   ├── next.config.ts
│   │   ├── tailwind.config.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── admin/                      # Management dashboard (SPA)
│   │   ├── app/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx            # redirect ke /dashboard
│   │   │   ├── (auth)/
│   │   │   │   └── login/
│   │   │   │       └── page.tsx
│   │   │   └── (dashboard)/
│   │   │       ├── layout.tsx      # sidebar + topbar
│   │   │       ├── dashboard/
│   │   │       │   └── page.tsx
│   │   │       ├── santri/
│   │   │       │   ├── page.tsx    # list + search
│   │   │       │   ├── [id]/
│   │   │       │   │   └── page.tsx
│   │   │       │   └── baru/
│   │   │       │       └── page.tsx
│   │   │       ├── kegiatan/
│   │   │       │   ├── page.tsx
│   │   │       │   ├── program/
│   │   │       │   ├── kelas/
│   │   │       │   ├── jadwal/
│   │   │       │   └── sesi/
│   │   │       ├── keuangan/
│   │   │       │   ├── page.tsx
│   │   │       │   ├── spp/
│   │   │       │   ├── donasi/
│   │   │       │   ├── pengeluaran/
│   │   │       │   └── laporan/
│   │   │       ├── proyek/
│   │   │       │   ├── page.tsx
│   │   │       │   └── [id]/
│   │   │       ├── konten/
│   │   │       │   ├── page.tsx
│   │   │       │   ├── pengumuman/
│   │   │       │   ├── galeri/
│   │   │       │   ├── landing/
│   │   │       │   └── artikel/
│   │   │       └── settings/
│   │   │           └── page.tsx
│   │   ├── components/
│   │   │   ├── sidebar.tsx
│   │   │   ├── topbar.tsx
│   │   │   ├── data-table.tsx
│   │   │   ├── stat-card.tsx
│   │   │   ├── attendance-grid.tsx
│   │   │   └── finance-chart.tsx
│   │   ├── next.config.ts
│   │   ├── tailwind.config.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   └── jamaah/                     # Parent/Donor PWA (mobile-first)
│       ├── app/
│       │   ├── layout.tsx
│       │   ├── page.tsx            # / (home)
│       │   ├── (auth)/
│       │   │   ├── login/
│       │   │   │   └── page.tsx
│       │   │   └── register/
│       │   │       └── page.tsx    # multi-step onboarding
│       │   ├── (main)/
│       │   │   ├── layout.tsx      # bottom-nav layout
│       │   │   ├── anak/
│       │   │   │   ├── page.tsx    # list anak
│       │   │   │   └── [id]/
│       │   │   │       ├── page.tsx    # detail anak
│       │   │   │       └── izin/
│       │   │   │           └── page.tsx
│       │   │   ├── bayar/
│       │   │   │   └── page.tsx
│       │   │   ├── masjid/
│       │   │   │   └── page.tsx
│       │   │   ├── notifikasi/
│       │   │   │   └── page.tsx
│       │   │   └── profil/
│       │   │       └── page.tsx
│       │   └── manifest.ts         # PWA manifest
│       ├── components/
│       │   ├── bottom-nav.tsx
│       │   ├── child-card.tsx
│       │   ├── attendance-calendar.tsx
│       │   ├── quran-progress.tsx
│       │   ├── payment-card.tsx
│       │   └── notification-item.tsx
│       ├── next.config.ts
│       ├── tailwind.config.ts
│       ├── tsconfig.json
│       └── package.json
│
├── packages/
│   ├── ui/                         # Shared UI components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── button.tsx
│   │   │   │   ├── card.tsx
│   │   │   │   ├── dialog.tsx
│   │   │   │   ├── input.tsx
│   │   │   │   ├── select.tsx
│   │   │   │   ├── table.tsx
│   │   │   │   ├── tabs.tsx
│   │   │   │   ├── badge.tsx
│   │   │   │   ├── avatar.tsx
│   │   │   │   ├── toast.tsx
│   │   │   │   ├── skeleton.tsx
│   │   │   │   ├── stat-card.tsx
│   │   │   │   ├── attendance-toggle.tsx
│   │   │   │   ├── quran-input.tsx
│   │   │   │   └── bottom-nav.tsx
│   │   │   ├── lib/
│   │   │   │   └── utils.ts
│   │   │   └── index.ts
│   │   ├── tailwind.config.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── db/                         # Database client & types
│   │   ├── src/
│   │   │   ├── client.ts           # Supabase client factory
│   │   │   ├── types.ts            # Generated types (supabase gen types)
│   │   │   ├── queries/
│   │   │   │   ├── profiles.ts
│   │   │   │   ├── children.ts
│   │   │   │   ├── attendance.ts
│   │   │   │   ├── quran-progress.ts
│   │   │   │   ├── study-progress.ts
│   │   │   │   ├── finance.ts
│   │   │   │   ├── programs.ts
│   │   │   │   ├── projects.ts
│   │   │   │   ├── content.ts
│   │   │   │   └── notifications.ts
│   │   │   ├── hooks/
│   │   │   │   ├── use-children.ts
│   │   │   │   ├── use-attendance.ts
│   │   │   │   ├── use-finance.ts
│   │   │   │   └── use-realtime.ts
│   │   │   └── index.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── auth/                       # Auth utilities
│   │   ├── src/
│   │   │   ├── client.ts           # Auth client wrapper
│   │   │   ├── middleware.ts       # Next.js middleware helper
│   │   │   ├── hooks/
│   │   │   │   ├── use-auth.ts
│   │   │   │   ├── use-session.ts
│   │   │   │   └── use-role.ts
│   │   │   ├── providers/
│   │   │   │   └── auth-provider.tsx
│   │   │   ├── guards/
│   │   │   │   ├── role-guard.tsx
│   │   │   │   └── auth-guard.tsx
│   │   │   ├── utils/
│   │   │   │   ├── pin.ts         # PIN hash/verify
│   │   │   │   └── otp.ts         # OTP generation
│   │   │   └── index.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── config/                     # Shared configurations
│   │   ├── eslint/
│   │   │   └── index.js
│   │   ├── tailwind/
│   │   │   └── base.config.ts
│   │   ├── typescript/
│   │   │   └── base.json
│   │   └── package.json
│   │
│   └── utils/                      # Shared utilities
│       ├── src/
│       │   ├── formatters/
│       │   │   ├── currency.ts     # Format Rupiah
│       │   │   ├── date.ts         # Format tanggal Indonesia
│       │   │   └── phone.ts        # Format nomor WA
│       │   ├── validators/
│       │   │   ├── phone.ts
│       │   │   ├── pin.ts
│       │   │   └── form.ts
│       │   ├── constants/
│       │   │   ├── roles.ts
│       │   │   ├── programs.ts
│       │   │   └── quran.ts        # Surah list, juz mapping
│       │   └── index.ts
│       ├── tsconfig.json
│       └── package.json
│
├── supabase/
│   ├── config.toml
│   ├── migrations/
│   │   ├── 00001_initial_schema.sql
│   │   ├── 00002_rls_policies.sql
│   │   ├── 00003_functions.sql
│   │   └── 00004_seed.sql
│   ├── functions/
│   │   ├── send-wa-otp/
│   │   │   └── index.ts
│   │   ├── generate-credentials/
│   │   │   └── index.ts
│   │   └── notify/
│   │       └── index.ts
│   └── seed.sql
│
├── turbo.json
├── pnpm-workspace.yaml
├── package.json
├── .env.example
├── .gitignore
├── README.md
└── docs/
    ├── PROJECT_SPEC.md
    ├── ARCHITECTURE.md
    ├── DATABASE_SCHEMA.md
    ├── AUTH_FLOW.md
    ├── SEO_STRATEGY.md
    ├── UI_UX_SPEC.md
    ├── SITEMAP.md
    └── API_SPEC.md
```

---

## Package Breakdown

### `@repo/ui` — Shared UI Components

Base shadcn/ui components yang di-extend dengan custom components spesifik platform:

```typescript
// packages/ui/src/index.ts
// shadcn/ui base components
export { Button } from './components/button'
export { Card, CardContent, CardHeader, CardTitle } from './components/card'
export { Dialog } from './components/dialog'
export { Input } from './components/input'
export { Table } from './components/table'
export { Tabs, TabsList, TabsTrigger, TabsContent } from './components/tabs'
export { Badge } from './components/badge'
export { Avatar } from './components/avatar'
export { Toast } from './components/toast'
export { Skeleton } from './components/skeleton'

// Custom platform components
export { StatCard } from './components/stat-card'
export { AttendanceToggle } from './components/attendance-toggle'
export { QuranInput } from './components/quran-input'
export { BottomNav } from './components/bottom-nav'
```

### `@repo/db` — Database Layer

Single source of truth untuk semua interaksi database:

```typescript
// packages/db/src/client.ts
import { createClient } from '@supabase/supabase-js'
import type { Database } from './types'

export function createSupabaseClient() {
  return createClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}

export function createSupabaseServerClient(cookieStore: any) {
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    { cookies: { /* ... */ } }
  )
}

// Type generation:
// pnpm supabase gen types typescript --project-id <id> > packages/db/src/types.ts
```

### `@repo/auth` — Authentication Layer

Handles semua auth logic termasuk phone-based login, OTP, PIN:

```typescript
// packages/auth/src/hooks/use-auth.ts
export function useAuth() {
  const supabase = useSupabaseClient()

  return {
    signInWithPhone: async (phone: string) => { /* ... */ },
    verifyOtp: async (phone: string, token: string) => { /* ... */ },
    verifyPin: async (pin: string) => { /* ... */ },
    signOut: async () => { /* ... */ },
    user: /* current user */,
    role: /* current role */,
    isLoading: boolean,
  }
}
```

### `@repo/config` — Shared Configurations

```typescript
// packages/config/tailwind/base.config.ts
export default {
  content: [], // di-extend per app
  theme: {
    extend: {
      colors: {
        primary: { /* emerald scale */ },
        secondary: { /* teal scale */ },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        arabic: ['Amiri', 'serif'],
      },
      borderRadius: {
        card: '12px',
        button: '16px',
      },
      spacing: {
        // 8px grid system
      },
    },
  },
}
```

### `@repo/utils` — Shared Utilities

```typescript
// packages/utils/src/formatters/currency.ts
export function formatRupiah(amount: number): string {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    minimumFractionDigits: 0,
  }).format(amount)
}

// packages/utils/src/constants/quran.ts
export const SURAH_LIST = [
  { number: 1, name: 'Al-Fatihah', arabicName: 'الفاتحة', totalAyat: 7 },
  // ... 114 surah
]
```

---

## How Apps Connect to Supabase

Semua 3 apps menggunakan **satu Supabase project** yang sama. Yang membedakan adalah:
1. **RLS policies** — membatasi akses berdasarkan role user
2. **Middleware** — setiap app punya auth middleware yang check role
3. **API keys** — semua pakai anon key yang sama, RLS yang membatasi

### Connection Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Landing    │     │    Admin     │     │   Jamaah     │
│  (SSG/ISR)   │     │    (SPA)     │     │   (PWA)      │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │
       │  anon key          │  anon key          │  anon key
       │  (public read)     │  (authenticated)   │  (authenticated)
       │                    │                    │
       └────────────────────┼────────────────────┘
                            │
                    ┌───────▼────────┐
                    │   Supabase     │
                    │  (Single       │
                    │   Project)     │
                    ├────────────────┤
                    │  Auth          │ ← Phone OTP + PIN
                    │  Database      │ ← PostgreSQL + RLS
                    │  Realtime      │ ← Live subscriptions
                    │  Storage       │ ← File uploads
                    │  Edge Functions│ ← OTP, notifications
                    └────────────────┘
```

### Access Patterns per App

| App | Auth Required | RLS Context | Data Access |
|-----|--------------|-------------|-------------|
| **Landing** | No | anon (public) | Read-only: programs, projects, articles, finance summary |
| **Admin** | Yes | superadmin/pengurus/pengajar | Full CRUD sesuai role |
| **Jamaah** | Yes | orangtua/donatur | Read anak sendiri, write izin/donasi |

### Server-Side vs Client-Side

```
Landing:
  - Server Components (RSC) → createSupabaseServerClient
  - Static Generation (SSG) → data fetched at build time
  - ISR → revalidate every N seconds

Admin:
  - Server Components for initial data
  - Client Components for interactive features
  - Real-time subscriptions for live data

Jamaah:
  - Server Components for initial data
  - Client Components for forms, interactions
  - Real-time subscriptions for notifications
  - Service Worker for PWA offline
```

---

## Deployment Topology

### Vercel Multi-Project Setup

```
GitHub Repo (monorepo)
        │
        ├──→ Vercel Project: masjid-landing
        │    ├── Root Directory: apps/landing
        │    ├── Domain: masjid.id / www.masjid.id
        │    ├── Build: turbo run build --filter=landing
        │    └── Framework: Next.js
        │
        ├──→ Vercel Project: masjid-admin
        │    ├── Root Directory: apps/admin
        │    ├── Domain: admin.masjid.id
        │    ├── Build: turbo run build --filter=admin
        │    └── Framework: Next.js
        │
        └──→ Vercel Project: masjid-jamaah
             ├── Root Directory: apps/jamaah
             ├── Domain: app.masjid.id
             ├── Build: turbo run build --filter=jamaah
             └── Framework: Next.js
```

### Supabase Deployment

```
Supabase Project: masjid-digital
├── Region: ap-southeast-1 (Singapore)
├── Plan: Pro (recommended for production)
├── Database: PostgreSQL 15+
├── Auth: Phone provider enabled
├── Storage Buckets:
│   ├── avatars (public)
│   ├── gallery (public)
│   ├── projects (public)
│   ├── receipts (private)
│   └── documents (private)
└── Edge Functions:
    ├── send-wa-otp
    ├── generate-credentials
    └── notify
```

---

## Environment Variables

### Shared (All Apps)
```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...        # Server-only, NEVER expose

# App URLs (for redirects, CORS)
NEXT_PUBLIC_LANDING_URL=https://masjid.id
NEXT_PUBLIC_ADMIN_URL=https://admin.masjid.id
NEXT_PUBLIC_JAMAAH_URL=https://app.masjid.id
```

### Landing-Specific
```env
# Analytics
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
NEXT_PUBLIC_GSC_VERIFICATION=google-site-verification=xxx

# ISR
REVALIDATE_SECRET=secret-for-on-demand-revalidation
```

### Admin-Specific
```env
# WhatsApp API (for sending OTP/notifications)
WA_API_URL=https://api.whatsapp.com/...
WA_API_TOKEN=xxx
```

### Jamaah-Specific
```env
# PWA
NEXT_PUBLIC_VAPID_PUBLIC_KEY=xxx
VAPID_PRIVATE_KEY=xxx
```

### Supabase Edge Functions
```env
WA_API_URL=https://api.whatsapp.com/...
WA_API_TOKEN=xxx
SUPABASE_SERVICE_ROLE_KEY=eyJ...
```

---

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo typecheck

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo test

  build:
    needs: [lint, typecheck, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo build

  deploy-supabase:
    needs: [build]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
      - run: supabase db push
      - run: supabase functions deploy
```

### Turborepo Pipeline Configuration

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": [".env"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "typecheck": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["^build"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

### pnpm Workspace Config

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

---

## Development Workflow

### Getting Started
```bash
# Clone & install
git clone <repo-url>
cd masjid-digital
pnpm install

# Setup environment
cp .env.example .env.local
# Edit .env.local dengan Supabase credentials

# Start Supabase locally
pnpm supabase start

# Run all apps in dev mode
pnpm dev

# Run specific app
pnpm dev --filter=landing
pnpm dev --filter=admin
pnpm dev --filter=jamaah
```

### Dev Ports
```
Landing:  http://localhost:3000
Admin:    http://localhost:3001
Jamaah:   http://localhost:3002
Supabase: http://localhost:54321 (Studio)
```

### Database Migrations
```bash
# Create new migration
pnpm supabase migration new <name>

# Apply migrations
pnpm supabase db push

# Generate types
pnpm supabase gen types typescript --local > packages/db/src/types.ts

# Reset database (development only)
pnpm supabase db reset
```

### Adding a Shared Package Dependency
```bash
# Add @repo/ui to landing app
cd apps/landing
pnpm add @repo/ui --workspace

# Import in component
import { Button, StatCard } from '@repo/ui'
```

---

## Error Handling & Monitoring

### Error Boundaries
- React Error Boundaries per page segment
- Global error handler di root layout
- Toast notifications untuk user-facing errors

### Logging (Future)
- Structured logging via Pino
- Error tracking via Sentry (optional)
- Supabase Dashboard untuk database monitoring

### Health Checks
- Supabase status endpoint
- Vercel deployment health
- Edge Function invocation logs
