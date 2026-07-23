# SEO STRATEGY - Platform Digital Masjid

## Overview

Strategi SEO yang agresif untuk landing app. Target: rank di halaman pertama Google untuk keyword terkait masjid, TPQ, donasi masjid, dan program ngaji di area lokal. Kombinasi technical SEO, structured data, content SEO, dan Core Web Vitals optimization.

---

## Technical SEO

### Rendering Strategy

| Halaman | Strategy | Revalidate | Keterangan |
|---------|----------|------------|------------|
| `/` (home) | SSG + ISR | 3600s (1 jam) | Content jarang berubah |
| `/tentang` | SSG | Build time | Hampir tidak pernah berubah |
| `/program` | SSG + ISR | 3600s | Update ketika program baru ditambah |
| `/program/[slug]` | SSG + ISR | 3600s | `generateStaticParams()` |
| `/proyek/[slug]` | SSG + ISR | 1800s (30 min) | Update lebih sering (progress) |
| `/keuangan` | SSG + ISR | 1800s | Data keuangan di-update berkala |
| `/artikel` | SSG + ISR | 900s (15 min) | New articles published periodically |
| `/artikel/[slug]` | SSG + ISR | 3600s | `generateStaticParams()` |
| `/daftar` | SSG | Build time | Static redirect page |
| `/legal/*` | SSG | Build time | Rarely changes |

### Implementation

```typescript
// apps/landing/app/artikel/[slug]/page.tsx

import { createSupabaseServerClient } from '@repo/db'

// Generate static paths at build time
export async function generateStaticParams() {
  const supabase = createSupabaseServerClient()
  const { data: articles } = await supabase
    .from('articles')
    .select('slug')
    .eq('status', 'published')

  return articles?.map((a) => ({ slug: a.slug })) ?? []
}

// ISR revalidation
export const revalidate = 3600

// Page component with metadata
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const article = await getArticle(params.slug)
  return {
    title: article.meta_title || article.title,
    description: article.meta_description || article.excerpt,
    openGraph: {
      title: article.title,
      description: article.excerpt,
      images: [article.cover_image_url],
      type: 'article',
      publishedTime: article.published_at,
    },
  }
}
```

### Sitemap Configuration

```typescript
// apps/landing/app/sitemap.ts

import { MetadataRoute } from 'next'
import { createSupabaseServerClient } from '@repo/db'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const supabase = createSupabaseServerClient()
  const baseUrl = process.env.NEXT_PUBLIC_LANDING_URL!

  // Static pages
  const staticPages = [
    { url: baseUrl, lastModified: new Date(), changeFrequency: 'daily' as const, priority: 1.0 },
    { url: `${baseUrl}/tentang`, lastModified: new Date(), changeFrequency: 'monthly' as const, priority: 0.8 },
    { url: `${baseUrl}/program`, lastModified: new Date(), changeFrequency: 'weekly' as const, priority: 0.9 },
    { url: `${baseUrl}/keuangan`, lastModified: new Date(), changeFrequency: 'daily' as const, priority: 0.8 },
    { url: `${baseUrl}/artikel`, lastModified: new Date(), changeFrequency: 'daily' as const, priority: 0.7 },
    { url: `${baseUrl}/daftar`, lastModified: new Date(), changeFrequency: 'monthly' as const, priority: 0.6 },
  ]

  // Dynamic: programs
  const { data: programs } = await supabase
    .from('programs')
    .select('slug, updated_at')
    .eq('is_public', true)

  const programPages = programs?.map((p) => ({
    url: `${baseUrl}/program/${p.slug}`,
    lastModified: new Date(p.updated_at),
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  })) ?? []

  // Dynamic: projects
  const { data: projects } = await supabase
    .from('projects')
    .select('slug, updated_at')
    .eq('is_public', true)

  const projectPages = projects?.map((p) => ({
    url: `${baseUrl}/proyek/${p.slug}`,
    lastModified: new Date(p.updated_at),
    changeFrequency: 'weekly' as const,
    priority: 0.7,
  })) ?? []

  // Dynamic: articles
  const { data: articles } = await supabase
    .from('articles')
    .select('slug, updated_at')
    .eq('status', 'published')

  const articlePages = articles?.map((a) => ({
    url: `${baseUrl}/artikel/${a.slug}`,
    lastModified: new Date(a.updated_at),
    changeFrequency: 'monthly' as const,
    priority: 0.6,
  })) ?? []

  // Legal pages
  const legalPages = [
    'syarat-ketentuan',
    'kebijakan-privasi',
    'kebijakan-donasi',
    'kebijakan-cookie',
  ].map((slug) => ({
    url: `${baseUrl}/legal/${slug}`,
    lastModified: new Date(),
    changeFrequency: 'yearly' as const,
    priority: 0.3,
  }))

  return [...staticPages, ...programPages, ...projectPages, ...articlePages, ...legalPages]
}
```

### Robots.txt

```typescript
// apps/landing/app/robots.ts

import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  const baseUrl = process.env.NEXT_PUBLIC_LANDING_URL!

  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/api/', '/_next/'],
      },
    ],
    sitemap: `${baseUrl}/sitemap.xml`,
  }
}
```

### Canonical URLs

Setiap halaman harus punya canonical URL untuk menghindari duplicate content:

```typescript
// apps/landing/app/layout.tsx
export const metadata: Metadata = {
  metadataBase: new URL(process.env.NEXT_PUBLIC_LANDING_URL!),
  alternates: {
    canonical: '/',
  },
}

// Per page
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  return {
    alternates: {
      canonical: `/artikel/${params.slug}`,
    },
  }
}
```

---

## Structured Data (JSON-LD)

### Organization (Home Page)

```typescript
// apps/landing/lib/structured-data.ts

export function getOrganizationSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': ['Organization', 'Mosque'],
    name: 'Masjid [Nama Masjid]',
    alternateName: '[Nama Alternatif]',
    url: process.env.NEXT_PUBLIC_LANDING_URL,
    logo: `${process.env.NEXT_PUBLIC_LANDING_URL}/logo.png`,
    image: `${process.env.NEXT_PUBLIC_LANDING_URL}/og-image.jpg`,
    description: 'Masjid [nama] - pusat kegiatan ibadah, pendidikan Al-Quran, dan pemberdayaan umat.',
    address: {
      '@type': 'PostalAddress',
      streetAddress: '[Alamat Jalan]',
      addressLocality: '[Kota]',
      addressRegion: '[Provinsi]',
      postalCode: '[Kode Pos]',
      addressCountry: 'ID',
    },
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+62-xxx-xxxx-xxxx',
      contactType: 'customer service',
      availableLanguage: 'Indonesian',
    },
    sameAs: [
      'https://www.instagram.com/masjidxxx',
      'https://www.facebook.com/masjidxxx',
      'https://www.youtube.com/@masjidxxx',
    ],
  }
}
```

### LocalBusiness (Tentang Page)

```typescript
export function getLocalBusinessSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'PlaceOfWorship',
    name: 'Masjid [Nama Masjid]',
    image: `${process.env.NEXT_PUBLIC_LANDING_URL}/masjid-photo.jpg`,
    url: `${process.env.NEXT_PUBLIC_LANDING_URL}/tentang`,
    telephone: '+62-xxx-xxxx-xxxx',
    address: {
      '@type': 'PostalAddress',
      streetAddress: '[Alamat]',
      addressLocality: '[Kota]',
      addressRegion: '[Provinsi]',
      postalCode: '[Kode Pos]',
      addressCountry: 'ID',
    },
    geo: {
      '@type': 'GeoCoordinates',
      latitude: -6.xxx,
      longitude: 106.xxx,
    },
    openingHoursSpecification: [
      {
        '@type': 'OpeningHoursSpecification',
        dayOfWeek: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'],
        opens: '04:00',
        closes: '22:00',
      },
    ],
  }
}
```

### NonprofitType + DonateAction (Keuangan Page)

```typescript
export function getDonationSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'DonateAction',
    name: 'Donasi ke Masjid [Nama]',
    description: 'Donasi untuk mendukung kegiatan dan pembangunan masjid',
    recipient: {
      '@type': 'Organization',
      name: 'Masjid [Nama Masjid]',
      '@id': process.env.NEXT_PUBLIC_LANDING_URL,
    },
    url: `${process.env.NEXT_PUBLIC_LANDING_URL}/keuangan`,
    target: {
      '@type': 'EntryPoint',
      urlTemplate: `${process.env.NEXT_PUBLIC_LANDING_URL}/keuangan`,
      actionPlatform: [
        'http://schema.org/DesktopWebPlatform',
        'http://schema.org/MobileWebPlatform',
      ],
    },
  }
}

export function getNonprofitSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'NGO',
    name: 'Masjid [Nama Masjid]',
    nonprofitStatus: 'https://schema.org/Nonprofit501c3',
    url: process.env.NEXT_PUBLIC_LANDING_URL,
    description: 'Lembaga keagamaan yang bergerak di bidang pendidikan Al-Quran dan pemberdayaan umat.',
  }
}
```

### Article (Detail Artikel)

```typescript
export function getArticleSchema(article: Article) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: article.title,
    description: article.excerpt,
    image: article.cover_image_url,
    datePublished: article.published_at,
    dateModified: article.updated_at,
    author: {
      '@type': 'Organization',
      name: 'Masjid [Nama Masjid]',
      url: process.env.NEXT_PUBLIC_LANDING_URL,
    },
    publisher: {
      '@type': 'Organization',
      name: 'Masjid [Nama Masjid]',
      logo: {
        '@type': 'ImageObject',
        url: `${process.env.NEXT_PUBLIC_LANDING_URL}/logo.png`,
      },
    },
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': `${process.env.NEXT_PUBLIC_LANDING_URL}/artikel/${article.slug}`,
    },
  }
}
```

### BreadcrumbList (All Pages)

```typescript
export function getBreadcrumbSchema(items: { name: string; url: string }[]) {
  return {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: item.name,
      item: item.url,
    })),
  }
}

// Usage:
// getBreadcrumbSchema([
//   { name: 'Beranda', url: 'https://masjid.id' },
//   { name: 'Program', url: 'https://masjid.id/program' },
//   { name: 'Tahfidz Al-Quran', url: 'https://masjid.id/program/tahfidz-al-quran' },
// ])
```

### Event (Program / Kegiatan)

```typescript
export function getEventSchema(program: Program) {
  return {
    '@context': 'https://schema.org',
    '@type': 'EducationEvent',
    name: program.name,
    description: program.description,
    image: program.image_url,
    organizer: {
      '@type': 'Organization',
      name: 'Masjid [Nama Masjid]',
      url: process.env.NEXT_PUBLIC_LANDING_URL,
    },
    location: {
      '@type': 'Place',
      name: 'Masjid [Nama Masjid]',
      address: {
        '@type': 'PostalAddress',
        streetAddress: '[Alamat]',
        addressLocality: '[Kota]',
        addressCountry: 'ID',
      },
    },
    url: `${process.env.NEXT_PUBLIC_LANDING_URL}/program/${program.slug}`,
    eventAttendanceMode: 'https://schema.org/OfflineEventAttendanceMode',
    isAccessibleForFree: true,
  }
}
```

### Injecting JSON-LD

```typescript
// apps/landing/app/layout.tsx or per page

export default function Page() {
  const orgSchema = getOrganizationSchema()
  const breadcrumb = getBreadcrumbSchema([...])

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(orgSchema) }}
      />
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(breadcrumb) }}
      />
      {/* page content */}
    </>
  )
}
```

---

## Meta Tags per Page

### Global (Layout)

```typescript
// apps/landing/app/layout.tsx

export const metadata: Metadata = {
  metadataBase: new URL(process.env.NEXT_PUBLIC_LANDING_URL!),
  title: {
    default: 'Masjid [Nama] - Pusat Pendidikan Al-Quran & Ibadah',
    template: '%s | Masjid [Nama]',
  },
  description: 'Platform digital Masjid [Nama] - pendidikan Al-Quran, program ngaji, donasi, dan transparansi keuangan.',
  keywords: ['masjid', 'TPQ', 'ngaji', 'Al-Quran', 'donasi masjid', 'pendidikan Islam', '[nama kota]'],
  authors: [{ name: 'Masjid [Nama]' }],
  openGraph: {
    type: 'website',
    locale: 'id_ID',
    siteName: 'Masjid [Nama]',
    images: [{ url: '/og-image.jpg', width: 1200, height: 630, alt: 'Masjid [Nama]' }],
  },
  twitter: {
    card: 'summary_large_image',
  },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
  verification: {
    google: process.env.NEXT_PUBLIC_GSC_VERIFICATION,
  },
}
```

### Per-Page Meta Tags

| Halaman | Title | Description Focus |
|---------|-------|-------------------|
| `/` | Masjid [Nama] - Pusat Pendidikan Al-Quran & Ibadah | Overview masjid, program unggulan, CTA daftar |
| `/tentang` | Tentang Kami \| Masjid [Nama] | Sejarah, visi misi, kepengurusan |
| `/program` | Program & Proyek \| Masjid [Nama] | Daftar program ngaji & belajar, proyek pembangunan |
| `/program/[slug]` | [Nama Program] \| Masjid [Nama] | Detail program, jadwal, pendaftaran |
| `/proyek/[slug]` | [Nama Proyek] \| Masjid [Nama] | Progress proyek, milestone, donasi |
| `/keuangan` | Donasi & Transparansi Keuangan \| Masjid [Nama] | Donasi online, laporan keuangan transparan |
| `/artikel` | Artikel & Kajian \| Masjid [Nama] | Blog, kajian Islam, berita masjid |
| `/artikel/[slug]` | [Judul Artikel] \| Masjid [Nama] | Konten artikel (excerpt) |
| `/daftar` | Daftar Sekarang \| Masjid [Nama] | Pendaftaran santri baru |
| `/legal/*` | [Judul Legal] \| Masjid [Nama] | Legal compliance |

---

## Core Web Vitals Targets

### Target Metrics

| Metric | Target | Strategy |
|--------|--------|----------|
| **LCP** (Largest Contentful Paint) | < 2.5s | SSG, optimized images, font preload |
| **FID** (First Input Delay) | < 100ms | Minimal JS, code splitting |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Fixed dimensions, font-display: swap |
| **INP** (Interaction to Next Paint) | < 200ms | Efficient event handlers |
| **TTFB** (Time to First Byte) | < 800ms | Edge caching, SSG |
| **FCP** (First Contentful Paint) | < 1.8s | Critical CSS inline |

### Optimization Strategies

#### Image Optimization
```typescript
// Next.js Image component dengan proper sizing
import Image from 'next/image'

<Image
  src={program.image_url}
  alt={program.name}
  width={400}
  height={300}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  loading="lazy"    // above-fold images use priority instead
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>

// Hero image - above fold
<Image
  src="/hero.jpg"
  alt="Masjid [Nama]"
  width={1200}
  height={600}
  priority          // Preload for LCP
  sizes="100vw"
/>
```

#### Font Optimization
```typescript
// apps/landing/app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',     // Prevent FOIT
  preload: true,
  variable: '--font-inter',
})

// Amiri font for Arabic text
const amiri = localFont({
  src: '../fonts/Amiri-Regular.woff2',
  display: 'swap',
  variable: '--font-amiri',
})
```

#### Code Splitting
```typescript
// Dynamic import untuk komponen heavy
import dynamic from 'next/dynamic'

const DonationForm = dynamic(() => import('@/components/donation-form'), {
  loading: () => <Skeleton className="h-[400px]" />,
  ssr: false, // Client-only component
})

const TransparencyChart = dynamic(() => import('@/components/transparency-chart'), {
  loading: () => <Skeleton className="h-[300px]" />,
})
```

#### Script Optimization
```typescript
// Analytics script - load after page interactive
import Script from 'next/script'

<Script
  src={`https://www.googletagmanager.com/gtag/js?id=${GA_ID}`}
  strategy="afterInteractive"
/>
```

---

## Crawlability

### Internal Linking Strategy

```
Home
├── Link ke /tentang (section "Tentang Kami")
├── Link ke /program (section "Program Unggulan")
├── Link ke /keuangan (section "Donasi")
├── Link ke /artikel (section "Artikel Terbaru")
└── Link ke /daftar (CTA button)

Program <-> Detail Program (breadcrumb + back link)
Program <-> Proyek (tab navigation)
Artikel List <-> Detail Artikel (breadcrumb + related articles)
Keuangan -> Proyek detail (link dari transparansi)
Footer -> Legal pages (semua halaman)
```

### Navigation Structure

**Navbar** (semua halaman):
```
Logo | Tentang | Program | Keuangan | Artikel | [Daftar ->]
```

**Footer** (semua halaman):
```
Tentang Masjid      Program         Informasi      Legal
├ Sejarah           ├ Ngaji         ├ Artikel       ├ Syarat & Ketentuan
├ Visi Misi         └ Belajar       └ Keuangan      ├ Kebijakan Privasi
└ Pengurus            Kelompok                       ├ Kebijakan Donasi
                                                     └ Kebijakan Cookie

Kontak
├ Alamat
├ WhatsApp
└ Social Media
```

### Heading Hierarchy

Setiap halaman harus punya satu `<h1>` dan heading hierarchy yang proper:

```html
<!-- /program -->
<h1>Program & Proyek Masjid [Nama]</h1>
  <h2>Program Ngaji</h2>
    <h3>Iqra</h3>
    <h3>Al-Quran</h3>
    <h3>Hafalan</h3>
  <h2>Belajar Kelompok</h2>
    <h3>Matematika</h3>
    <h3>Bahasa Indonesia</h3>
  <h2>Proyek Pembangunan</h2>
    <h3>[Nama Proyek]</h3>
```

---

## Content SEO

### Artikel Strategy

Target keyword clusters:

| Cluster | Example Keywords | Content Type |
|---------|-----------------|--------------|
| Ngaji | cara belajar ngaji, tips hafalan quran, keutamaan membaca quran | How-to, listicle |
| Parenting Islami | mendidik anak sholeh, tips parenting islami, anak rajin ngaji | How-to, tips |
| Kegiatan Masjid | kegiatan masjid, program TPQ, jadwal pengajian | News, report |
| Donasi | keutamaan sedekah, pahala infaq, cara donasi masjid online | Educational |
| Ramadan | persiapan ramadan, jadwal imsakiyah, tips puasa anak | Seasonal |

### Content Calendar
- Minimum 2 artikel per minggu
- Mix: 60% evergreen, 30% news/update, 10% seasonal
- Setiap artikel: 800-1500 kata, internal links, 1 featured image
- Meta description: 150-160 characters

### URL Structure
```
/artikel/tips-mengajarkan-anak-membaca-al-quran
/artikel/keutamaan-shodaqoh-di-bulan-ramadan
/artikel/laporan-kegiatan-pesantren-kilat-2025
```

---

## Monitoring & Analytics

### Google Search Console (GSC)
- Verification via meta tag
- Submit sitemap.xml
- Monitor:
  - Coverage (indexed pages, errors)
  - Performance (clicks, impressions, CTR, position)
  - Core Web Vitals (field data)
  - Mobile usability

### Google Analytics 4 (GA4)
- Track pageviews, sessions, users
- Custom events:
  - `donation_click` -- klik tombol donasi
  - `registration_start` -- mulai pendaftaran
  - `registration_complete` -- selesai daftar
  - `article_read` -- baca artikel > 30s
  - `program_view` -- lihat detail program
  - `project_view` -- lihat detail proyek

### Implementation

```typescript
// apps/landing/lib/analytics.ts

export function trackEvent(name: string, params?: Record<string, any>) {
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('event', name, params)
  }
}

// Usage:
trackEvent('donation_click', {
  category: 'infaq',
  value: 100000,
  currency: 'IDR',
})
```

### Performance Monitoring
- Vercel Analytics (built-in) -- Real User Monitoring
- Web Vitals reporting:

```typescript
// apps/landing/app/layout.tsx

export function reportWebVitals(metric: NextWebVitalsMetric) {
  // Send to analytics
  trackEvent('web_vitals', {
    metric_name: metric.name,
    metric_value: Math.round(metric.value),
    metric_id: metric.id,
  })
}
```

### SEO Audit Checklist (Monthly)

- [ ] Check GSC for crawl errors
- [ ] Review Core Web Vitals scores
- [ ] Check for broken links (internal & external)
- [ ] Verify structured data (Google Rich Results Test)
- [ ] Review sitemap completeness
- [ ] Check robots.txt accessibility
- [ ] Monitor keyword rankings
- [ ] Review mobile usability issues
- [ ] Check page speed (PageSpeed Insights)
- [ ] Review 404 pages and redirects
- [ ] Ensure all images have alt text
- [ ] Verify canonical URLs are correct
- [ ] Check for duplicate content
- [ ] Review meta descriptions coverage
- [ ] Monitor backlink profile

---

## Advanced SEO Tactics

### Local SEO
- Google Business Profile (jika applicable)
- NAP (Name, Address, Phone) consistency
- Local keyword targeting: "[program] di [kota]"
- LocalBusiness schema markup

### Social Sharing Optimization
```typescript
// OG Image generation (dynamic)
// apps/landing/app/api/og/route.tsx

import { ImageResponse } from 'next/og'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const title = searchParams.get('title') ?? 'Masjid [Nama]'

  return new ImageResponse(
    (
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        justifyContent: 'center',
        width: '100%',
        height: '100%',
        background: 'linear-gradient(135deg, #059669, #0d9488)',
        color: 'white',
        fontSize: 48,
        fontWeight: 700,
        padding: 40,
      }}>
        <div>{title}</div>
      </div>
    ),
    { width: 1200, height: 630 }
  )
}
```

### Accessibility = SEO
- Semantic HTML (`<article>`, `<nav>`, `<main>`, `<aside>`, `<footer>`)
- Proper alt text untuk semua gambar
- ARIA labels untuk interactive elements
- Skip navigation link
- Language attribute: `<html lang="id">`
