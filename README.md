# Masjid Digital Platform

Platform digital terintegrasi untuk pengelolaan kegiatan masjid -- manajemen santri, program ngaji & belajar kelompok, keuangan (SPP/donasi), transparansi proyek, dan komunikasi dengan jamaah.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Monorepo | Turborepo + pnpm workspaces |
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS + shadcn/ui |
| Backend | Supabase (single project) |
| Database | PostgreSQL + RLS |
| Auth | Phone-based (WhatsApp OTP + 6-digit PIN) |
| Realtime | Supabase Realtime |
| Storage | Supabase Storage |
| Edge Functions | Supabase Edge Functions (Deno) |
| Deployment | Vercel (per app) |
| CI/CD | GitHub Actions |

---

## Applications

| App | Domain | Target User | Deskripsi |
|-----|--------|-------------|-----------|
| **Landing** | `masjid.id` | Publik | Website SSG+ISR, SEO-optimized. Profil masjid, program, donasi, artikel. |
| **Admin** | `admin.masjid.id` | Pengurus & Pengajar | Dashboard SPA. Manajemen santri, kegiatan, keuangan, konten. |
| **Jamaah** | `app.masjid.id` | Orang Tua & Donatur | PWA mobile-first. Lihat progress anak, bayar SPP, donasi, notifikasi. |

---

## Monorepo Structure

```
masjid-digital/
├── apps/
│   ├── landing/          # Public website (SSG+ISR)
│   ├── admin/            # Management dashboard (SPA)
│   └── jamaah/           # Parent/Donor app (PWA)
├── packages/
│   ├── ui/               # Shared UI components (shadcn/ui + custom)
│   ├── db/               # Supabase client, types, queries
│   ├── auth/             # Auth utilities, hooks, middleware
│   ├── config/           # Shared configs (Tailwind, ESLint, TS)
│   └── utils/            # Helpers, formatters, validators
├── supabase/
│   ├── migrations/       # SQL migrations
│   ├── functions/        # Edge Functions
│   └── seed.sql          # Seed data
├── docs/                 # Documentation
├── turbo.json
├── pnpm-workspace.yaml
└── package.json
```

---

## Documentation

| Document | Deskripsi |
|----------|-----------|
| [PROJECT_SPEC.md](docs/PROJECT_SPEC.md) | Full project specification: goals, roles, features, user flows, tech stack, phase plan |
| [ARCHITECTURE.md](docs/ARCHITECTURE.md) | System architecture: monorepo structure, packages, deployment topology, CI/CD |
| [DATABASE_SCHEMA.md](docs/DATABASE_SCHEMA.md) | Complete SQL schema: all tables, enums, RLS policies, indexes, triggers |
| [AUTH_FLOW.md](docs/AUTH_FLOW.md) | Auth documentation: phone-based auth, OTP, PIN management, registration flows, RBAC |
| [SEO_STRATEGY.md](docs/SEO_STRATEGY.md) | SEO strategy: technical SEO, structured data (JSON-LD), meta tags, Core Web Vitals |
| [UI_UX_SPEC.md](docs/UI_UX_SPEC.md) | Design system: colors, typography, spacing, components, wireframes, dark mode |
| [SITEMAP.md](docs/SITEMAP.md) | Complete page map for all 3 apps with route descriptions and rendering strategy |
| [API_SPEC.md](docs/API_SPEC.md) | API specification: Supabase queries, Edge Functions, Realtime, storage buckets |

---

## Quick Start

### Prerequisites
- Node.js 18+
- pnpm 8+
- Supabase CLI
- Git

### Setup

```bash
# Clone repository
git clone <repo-url>
cd masjid-digital

# Install dependencies
pnpm install

# Copy environment variables
cp .env.example .env.local
# Edit .env.local with your Supabase credentials

# Start Supabase locally
pnpm supabase start

# Apply database migrations
pnpm supabase db push

# Generate TypeScript types
pnpm supabase gen types typescript --local > packages/db/src/types.ts

# Start development (all apps)
pnpm dev
```

### Development URLs

```
Landing:   http://localhost:3000
Admin:     http://localhost:3001
Jamaah:    http://localhost:3002
Supabase:  http://localhost:54321 (Studio)
```

### Useful Commands

```bash
# Run specific app
pnpm dev --filter=landing
pnpm dev --filter=admin
pnpm dev --filter=jamaah

# Build all apps
pnpm build

# Lint all packages
pnpm lint

# Type check
pnpm typecheck

# Run tests
pnpm test

# Database commands
pnpm supabase migration new <name>    # Create migration
pnpm supabase db push                 # Apply migrations
pnpm supabase db reset                # Reset database (dev only)

# Deploy Edge Functions
pnpm supabase functions deploy
```

---

## Roles

| Role | Deskripsi |
|------|-----------|
| `superadmin` | Full system access, user management, configuration |
| `pengurus` | Financial management, program management, content |
| `pengajar` | Session management, attendance, student progress |
| `orangtua` | View child progress, pay SPP, submit leave requests |
| `donatur` | Make donations, view project transparency |

---

## Key Features

- **Santri Management** -- CRUD anak + orang tua, class enrollment
- **Attendance** -- Hybrid: teacher session-based input + parent leave requests
- **Quran Progress** -- Track Iqra, Al-Quran, Hafalan, Doa, Hadits progress
- **Study Progress** -- Track academic subject progress (belajar kelompok)
- **Digital Report Cards** -- Auto-generated semester reports
- **Finance** -- SPP invoicing, donation tracking, expense recording
- **Financial Transparency** -- Public financial summaries on landing page
- **Project Management** -- Construction/activity projects with milestones
- **Content Management** -- Announcements, gallery, articles, landing page CMS
- **Real-time Notifications** -- In-app + WhatsApp notifications
- **SEO-Optimized Landing** -- SSG/ISR, structured data, Core Web Vitals

---

## License

MIT
