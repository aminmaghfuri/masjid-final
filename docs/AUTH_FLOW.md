# AUTH FLOW - Platform Digital Masjid

## Overview

Platform ini menggunakan **phone-based authentication** tanpa email. Login via nomor WhatsApp + 6-digit PIN. OTP dikirim via WhatsApp API. Semua auth dihandle oleh Supabase Auth dengan custom logic via Edge Functions.

---

## Auth Strategy

### Kenapa Phone-Based?
1. **Target user** — Orang tua santri masjid, mayoritas lebih familiar dengan WhatsApp daripada email
2. **Simplicity** — Satu channel komunikasi (WhatsApp) untuk auth + notifikasi
3. **Trust** — Nomor WA sudah terverifikasi secara natural
4. **No password fatigue** — PIN 6 digit lebih mudah diingat

### Auth Components

| Component | Provider | Detail |
|-----------|----------|--------|
| Phone Auth | Supabase Auth | `signInWithOtp({ phone })` |
| OTP Delivery | WhatsApp Business API | Via Edge Function `send-wa-otp` |
| PIN Storage | PostgreSQL | bcrypt hashed di `profiles.pin_hash` |
| Session | Supabase Auth | JWT token, auto-refresh |
| Role Check | Custom RLS + middleware | `profiles.roles` array |

---

## Registration Flow

### Flow 1: Self-Register (Orang Tua via Landing)

User datang ke landing page, klik "Daftar", redirect ke jamaah app `/register`.

```
┌─────────────────────────────────────────────────────────┐
│                    REGISTRATION FLOW                     │
│                   (Self-Register)                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Step 1: Input Data Diri                                │
│  ┌──────────────────────────┐                           │
│  │  Nama Lengkap: [_______] │                           │
│  │  No. WhatsApp: [_______] │                           │
│  │         [Lanjut]         │                           │
│  └──────────────────────────┘                           │
│              │                                           │
│              ▼                                           │
│  Step 2: Verifikasi OTP                                 │
│  ┌──────────────────────────┐                           │
│  │  Kode OTP dikirim ke     │                           │
│  │  WhatsApp 0812-xxxx-xxxx │                           │
│  │                          │                           │
│  │  [_] [_] [_] [_] [_] [_]│                           │
│  │                          │                           │
│  │  Kirim ulang (59s)       │                           │
│  └──────────────────────────┘                           │
│              │                                           │
│              ▼                                           │
│  Step 3: Set PIN                                        │
│  ┌──────────────────────────┐                           │
│  │  Buat PIN 6 digit        │                           │
│  │  [_] [_] [_] [_] [_] [_]│                           │
│  │                          │                           │
│  │  Konfirmasi PIN          │                           │
│  │  [_] [_] [_] [_] [_] [_]│                           │
│  │         [Daftar]         │                           │
│  └──────────────────────────┘                           │
│              │                                           │
│              ▼                                           │
│  Step 4: Berhasil!                                      │
│  ┌──────────────────────────┐                           │
│  │  Pendaftaran berhasil!   │                           │
│  │                          │                           │
│  │  Selanjutnya tambahkan   │                           │
│  │  data anak Anda          │                           │
│  │   [Tambah Anak]          │                           │
│  └──────────────────────────┘                           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Technical Flow (Self-Register)

```typescript
// Step 1: Collect user info
const { fullName, phone } = formData

// Step 2: Send OTP via Supabase Auth + WhatsApp
const { error } = await supabase.auth.signInWithOtp({
  phone: formatPhone(phone), // +62812xxxxxxxx
  options: {
    data: {
      full_name: fullName,
      roles: ['orangtua'],
    },
    channel: 'whatsapp', // Supabase sends via configured provider
  },
})
// Alternatively, call Edge Function for custom WA template:
// await supabase.functions.invoke('send-wa-otp', { body: { phone, fullName } })

// Step 3: Verify OTP
const { data, error } = await supabase.auth.verifyOtp({
  phone: formatPhone(phone),
  token: otpCode, // 6-digit code from WhatsApp
  type: 'sms',    // Supabase uses 'sms' type even for WhatsApp
})

// Step 4: Set PIN (after successful OTP verification, user is now logged in)
const pinHash = await bcrypt.hash(pin, 12)
await supabase
  .from('profiles')
  .update({ pin_hash: pinHash })
  .eq('id', data.user.id)

// Step 5: Redirect to /anak to add children
router.push('/anak')
```

### Flow 2: Admin-Register (Admin Mendaftarkan Anak + Ortu)

Admin membuat data anak dan orang tua secara langsung dari dashboard.

```
┌─────────────────────────────────────────────────────────┐
│                 ADMIN REGISTRATION FLOW                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Admin Dashboard /santri -> Tambah Santri Baru          │
│                                                          │
│  Step 1: Input Data Anak                                │
│  ┌──────────────────────────────────┐                   │
│  │  Nama Lengkap : [______________] │                   │
│  │  Nama Panggilan: [____________] │                    │
│  │  Tanggal Lahir : [__/__/____]   │                   │
│  │  Jenis Kelamin : [v Laki-laki]  │                   │
│  │  Sekolah       : [______________]│                   │
│  │  Kelas         : [______________]│                   │
│  └──────────────────────────────────┘                   │
│              │                                           │
│              ▼                                           │
│  Step 2: Input Data Orang Tua                           │
│  ┌──────────────────────────────────┐                   │
│  │  * Orang tua sudah terdaftar     │                   │
│  │    -> [Cari berdasarkan No WA]   │                   │
│  │                                   │                   │
│  │  * Daftarkan orang tua baru      │                   │
│  │    Nama : [__________________]    │                   │
│  │    WA   : [__________________]    │                   │
│  │    Hubungan : [v Orang Tua]       │                   │
│  └──────────────────────────────────┘                   │
│              │                                           │
│              ▼                                           │
│  Step 3: Generate Credentials                           │
│  ┌──────────────────────────────────┐                   │
│  │  Credentials dibuat otomatis:     │                   │
│  │                                   │                   │
│  │  No. WhatsApp : 0812-3456-7890   │                   │
│  │  PIN Sementara: 847291            │                   │
│  │                                   │                   │
│  │  [Copy]  [Kirim via WA]          │                   │
│  └──────────────────────────────────┘                   │
│              │                                           │
│              ▼                                           │
│  System: Edge Function 'generate-credentials'           │
│  -> Create auth.user via service role                   │
│  -> Create profile + child + parent_child records       │
│  -> Create invitation record                            │
│  -> Send credentials via WhatsApp                       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Technical Flow (Admin-Register)

```typescript
// Edge Function: generate-credentials

import { createClient } from '@supabase/supabase-js'
import { serve } from 'https://deno.land/std/http/server.ts'
import * as bcrypt from 'https://deno.land/x/bcrypt/mod.ts'

serve(async (req) => {
  const { childData, parentData } = await req.json()

  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')! // service role for admin operations
  )

  // 1. Generate temporary PIN
  const tempPin = Math.floor(100000 + Math.random() * 900000).toString()
  const pinHash = await bcrypt.hash(tempPin)

  // 2. Create auth user (service role)
  const { data: authUser, error: authError } = await supabase.auth.admin.createUser({
    phone: formatPhone(parentData.phone),
    phone_confirm: true, // Skip OTP for admin-created users
    user_metadata: {
      full_name: parentData.name,
      roles: ['orangtua'],
    },
  })

  if (authError) throw authError

  // 3. Update profile with PIN
  await supabase
    .from('profiles')
    .update({ pin_hash: pinHash })
    .eq('id', authUser.user.id)

  // 4. Create child record
  const { data: child } = await supabase
    .from('children')
    .insert(childData)
    .select()
    .single()

  // 5. Link parent to child
  await supabase
    .from('parent_children')
    .insert({
      parent_id: authUser.user.id,
      child_id: child.id,
      relation: parentData.relation || 'orangtua',
      is_primary: true,
    })

  // 6. Create invitation record
  await supabase
    .from('invitations')
    .insert({
      phone: parentData.phone,
      child_id: child.id,
      generated_pin: pinHash,
      invited_by: req.headers.get('x-admin-id'),
    })

  // 7. Send credentials via WhatsApp
  await sendWhatsApp(parentData.phone, {
    template: 'registration_credentials',
    parameters: {
      name: parentData.name,
      childName: childData.full_name,
      pin: tempPin,
      loginUrl: `${Deno.env.get('JAMAAH_URL')}/login`,
    },
  })

  return new Response(JSON.stringify({ success: true, tempPin }))
})
```

---

## Login Flow

### Standard Login (WA + PIN)

```
┌─────────────────────────────────────────────────────────┐
│                      LOGIN FLOW                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Step 1: Input Nomor WhatsApp                           │
│  ┌──────────────────────────┐                           │
│  │  Masuk ke Akun Anda      │                           │
│  │                          │                           │
│  │  No. WhatsApp:           │                           │
│  │  [+62 ____________]      │                           │
│  │                          │                           │
│  │      [Lanjut]            │                           │
│  └──────────────────────────┘                           │
│              │                                           │
│              ▼                                           │
│  Step 2: Kirim OTP                                      │
│  -> System mengirim 6-digit OTP ke WhatsApp             │
│  -> OTP berlaku 5 menit                                 │
│  -> Rate limit: max 3 OTP per 15 menit                  │
│              │                                           │
│              ▼                                           │
│  Step 3: Input OTP                                      │
│  ┌──────────────────────────┐                           │
│  │  Masukkan Kode OTP       │                           │
│  │                          │                           │
│  │  [_] [_] [_] [_] [_] [_]│                           │
│  │                          │                           │
│  │  Kirim ulang (59s)       │                           │
│  └──────────────────────────┘                           │
│              │                                           │
│              ▼                                           │
│  Step 4: Input PIN                                      │
│  ┌──────────────────────────┐                           │
│  │  Masukkan PIN Anda       │                           │
│  │                          │                           │
│  │  [_] [_] [_] [_] [_] [_]│                           │
│  │                          │                           │
│  │  Lupa PIN?               │                           │
│  │      [Masuk]             │                           │
│  └──────────────────────────┘                           │
│              │                                           │
│              ▼                                           │
│  Logged in! -> Redirect berdasarkan role:               │
│  - orangtua/donatur -> Jamaah /                         │
│  - pengajar -> Admin /kegiatan                          │
│  - pengurus/superadmin -> Admin /dashboard              │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Technical Flow (Login)

```typescript
// Step 1-2: Send OTP
async function handleSendOtp(phone: string) {
  // Check if user exists
  const { data: profile } = await supabase
    .from('profiles')
    .select('id, is_active')
    .eq('phone', formatPhone(phone))
    .single()

  if (!profile) {
    throw new Error('Nomor tidak terdaftar')
  }
  if (!profile.is_active) {
    throw new Error('Akun dinonaktifkan')
  }

  // Send OTP via Supabase Auth
  const { error } = await supabase.auth.signInWithOtp({
    phone: formatPhone(phone),
  })

  if (error) throw error
}

// Step 3: Verify OTP
async function handleVerifyOtp(phone: string, otp: string) {
  const { data, error } = await supabase.auth.verifyOtp({
    phone: formatPhone(phone),
    token: otp,
    type: 'sms',
  })

  if (error) throw error
  // OTP verified, but we still need PIN verification
  // Store session temporarily
  return data.session
}

// Step 4: Verify PIN
async function handleVerifyPin(pin: string) {
  const { data: profile } = await supabase
    .from('profiles')
    .select('pin_hash, roles')
    .eq('id', supabase.auth.getUser().id)
    .single()

  const isValid = await bcrypt.compare(pin, profile.pin_hash)
  if (!isValid) {
    throw new Error('PIN salah')
  }

  // PIN valid -> redirect based on role
  return profile.roles
}
```

### First Login (Admin-Created Account)

Ketika admin membuat akun, ortu mendapat credentials via WhatsApp. Login pertama kali sedikit berbeda:

```
Login biasa (OTP + temporary PIN)
  -> System detect: first login (invitation status = 'pending')
  -> Force ganti PIN
  -> Set PIN baru
  -> Invitation status = 'accepted'
  -> Continue ke home
```

```typescript
// After successful login, check if first login
async function checkFirstLogin() {
  const user = await supabase.auth.getUser()

  const { data: invitation } = await supabase
    .from('invitations')
    .select('*')
    .eq('phone', user.phone)
    .eq('status', 'pending')
    .single()

  if (invitation) {
    // Force PIN change
    router.push('/profil/ganti-pin?first=true')

    // After PIN change, update invitation
    await supabase
      .from('invitations')
      .update({
        status: 'accepted',
        accepted_at: new Date().toISOString(),
      })
      .eq('id', invitation.id)
  }
}
```

---

## OTP Mechanism

### OTP Configuration

| Parameter | Value | Keterangan |
|-----------|-------|------------|
| Length | 6 digit | Numeric only |
| Expiry | 5 menit | Setelah itu expired |
| Max attempts | 3x | Per OTP code |
| Rate limit | 3 OTP / 15 menit | Per phone number |
| Resend cooldown | 60 detik | Sebelum bisa kirim ulang |
| Channel | WhatsApp | Via WhatsApp Business API |

### WhatsApp OTP Template

```
Assalamu'alaikum, {nama}

Kode OTP Anda untuk login ke Masjid Digital:

*{OTP_CODE}*

Kode ini berlaku selama 5 menit.
Jangan bagikan kode ini kepada siapapun.

Jazakallahu khairan.
```

### Edge Function: send-wa-otp

```typescript
// supabase/functions/send-wa-otp/index.ts

import { serve } from 'https://deno.land/std/http/server.ts'

serve(async (req) => {
  const { phone, otp, name } = await req.json()

  const waApiUrl = Deno.env.get('WA_API_URL')!
  const waApiToken = Deno.env.get('WA_API_TOKEN')!

  const response = await fetch(waApiUrl, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${waApiToken}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      messaging_product: 'whatsapp',
      to: phone,
      type: 'template',
      template: {
        name: 'otp_verification',
        language: { code: 'id' },
        components: [
          {
            type: 'body',
            parameters: [
              { type: 'text', text: name },
              { type: 'text', text: otp },
            ],
          },
          {
            type: 'button',
            sub_type: 'url',
            index: 0,
            parameters: [
              { type: 'text', text: otp },
            ],
          },
        ],
      },
    }),
  })

  const result = await response.json()
  return new Response(JSON.stringify(result))
})
```

---

## PIN Management

### PIN Rules
- Panjang: 6 digit (numeric only)
- Disimpan sebagai bcrypt hash di `profiles.pin_hash`
- Tidak boleh sama dengan 6 digit berurutan (123456, 654321)
- Tidak boleh semua digit sama (111111, 222222)

### Set PIN (Registration)

```typescript
async function setPin(pin: string, confirmPin: string) {
  // Validation
  if (pin !== confirmPin) throw new Error('PIN tidak cocok')
  if (pin.length !== 6) throw new Error('PIN harus 6 digit')
  if (/^(\d)\1{5}$/.test(pin)) throw new Error('PIN tidak boleh semua digit sama')
  if ('0123456789'.includes(pin) || '9876543210'.includes(pin)) {
    throw new Error('PIN tidak boleh berurutan')
  }

  const pinHash = await bcrypt.hash(pin, 12)

  await supabase
    .from('profiles')
    .update({ pin_hash: pinHash })
    .eq('id', user.id)
}
```

### Ganti PIN

```typescript
async function changePin(oldPin: string, newPin: string, confirmPin: string) {
  // Verify old PIN
  const { data: profile } = await supabase
    .from('profiles')
    .select('pin_hash')
    .eq('id', user.id)
    .single()

  const isValid = await bcrypt.compare(oldPin, profile.pin_hash)
  if (!isValid) throw new Error('PIN lama salah')

  // Set new PIN (same validation as above)
  await setPin(newPin, confirmPin)
}
```

### Lupa PIN

Jika user lupa PIN, flow-nya:

```
Klik "Lupa PIN?" di halaman login
  -> Kirim ulang OTP ke WhatsApp
  -> Verify OTP
  -> Set PIN baru (tanpa perlu PIN lama)
  -> Login otomatis
```

```typescript
async function resetPin(phone: string) {
  // 1. Send OTP
  await supabase.auth.signInWithOtp({ phone })

  // 2. After OTP verification, allow PIN reset without old PIN
  // This is handled by the UI flow -- after OTP verify,
  // show "Set PIN Baru" form instead of "Masukkan PIN"
}
```

---

## Credential Generation (Admin-Created)

### Generate Credentials Edge Function

```typescript
// supabase/functions/generate-credentials/index.ts

function generatePin(): string {
  let pin: string
  do {
    pin = Math.floor(100000 + Math.random() * 900000).toString()
  } while (
    /^(\d)\1{5}$/.test(pin) ||              // Tidak boleh semua sama
    '0123456789'.includes(pin) ||            // Tidak boleh ascending
    '9876543210'.includes(pin)               // Tidak boleh descending
  )
  return pin
}

// WhatsApp message untuk credentials
const credentialsMessage = `
Assalamu'alaikum, {nama}

Anda telah didaftarkan di platform Masjid Digital.

Anak terdaftar: {nama_anak}

Untuk masuk, gunakan informasi berikut:
- Nomor WhatsApp: {phone}
- PIN Sementara: {pin}
- Link Login: {login_url}

Anda akan diminta mengganti PIN saat login pertama kali.

Jazakallahu khairan.
`
```

---

## Session Management

### Session Configuration

| Parameter | Value |
|-----------|-------|
| Session type | JWT (Supabase default) |
| Access token expiry | 1 jam |
| Refresh token expiry | 30 hari |
| Auto-refresh | Yes (via Supabase client) |

### Middleware (Next.js)

```typescript
// packages/auth/src/middleware.ts

import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function authMiddleware(
  req: NextRequest,
  allowedRoles?: string[]
) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req, res })

  // Refresh session
  const { data: { session } } = await supabase.auth.getSession()

  if (!session) {
    return NextResponse.redirect(new URL('/login', req.url))
  }

  // Check roles if specified
  if (allowedRoles) {
    const { data: profile } = await supabase
      .from('profiles')
      .select('roles')
      .eq('id', session.user.id)
      .single()

    const hasRole = profile?.roles.some((r: string) => allowedRoles.includes(r))
    if (!hasRole) {
      return NextResponse.redirect(new URL('/unauthorized', req.url))
    }
  }

  return res
}
```

### Per-App Middleware

```typescript
// apps/admin/middleware.ts
import { authMiddleware } from '@repo/auth'

export async function middleware(req: NextRequest) {
  return authMiddleware(req, ['superadmin', 'pengurus', 'pengajar'])
}

export const config = {
  matcher: ['/(dashboard|santri|kegiatan|keuangan|proyek|konten|settings)/:path*'],
}

// apps/jamaah/middleware.ts
import { authMiddleware } from '@repo/auth'

export async function middleware(req: NextRequest) {
  // /register dan /login tidak perlu auth
  if (req.nextUrl.pathname.startsWith('/register') ||
      req.nextUrl.pathname.startsWith('/login')) {
    return NextResponse.next()
  }

  return authMiddleware(req, ['orangtua', 'donatur'])
}

export const config = {
  matcher: ['/((?!register|login|_next|api|favicon.ico).*)'],
}
```

---

## Role-Based Access Control (RBAC)

### Role Matrix

| Feature | superadmin | pengurus | pengajar | orangtua | donatur |
|---------|:----------:|:--------:|:--------:|:--------:|:-------:|
| Dashboard stats | Full | Full | Limited | - | - |
| Manage users | CRUD | Read | - | - | - |
| Manage santri | CRUD | CRUD | Read | Own children | - |
| Manage programs | CRUD | CRUD | Read | - | - |
| Create sessions | CRUD | CRUD | Own classes | - | - |
| Input attendance | All | All | Own sessions | - | - |
| Input quran progress | All | All | Own sessions | - | - |
| Input study progress | All | All | Own sessions | - | - |
| View progress | All | All | Own classes | Own children | - |
| Generate rapor | All | All | - | - | - |
| Manage finance | CRUD | CRUD | - | - | - |
| View invoices | All | All | - | Own | - |
| Make donation | - | - | - | Yes | Yes |
| Manage projects | CRUD | CRUD | - | - | - |
| Manage content | CRUD | CRUD | - | - | - |
| System settings | CRUD | Read | - | - | - |
| Submit leave request | - | - | - | Yes | - |
| View notifications | All | All | Own | Own | Own |

### Client-Side Role Guard

```typescript
// packages/auth/src/guards/role-guard.tsx

'use client'

import { useRole } from '../hooks/use-role'

interface RoleGuardProps {
  children: React.ReactNode
  roles: string[]
  fallback?: React.ReactNode
}

export function RoleGuard({ children, roles, fallback = null }: RoleGuardProps) {
  const { userRoles, isLoading } = useRole()

  if (isLoading) return <div>Loading...</div>

  const hasAccess = userRoles.some(r => roles.includes(r))

  if (!hasAccess) return fallback

  return <>{children}</>
}

// Usage:
// <RoleGuard roles={['superadmin', 'pengurus']}>
//   <AdminOnlyFeature />
// </RoleGuard>
```

---

## Security Considerations

### Rate Limiting
```
OTP requests:     3 per 15 minutes per phone
PIN attempts:     5 per 15 minutes per user
Login attempts:   10 per hour per IP
API requests:     100 per minute per user
```

### Token Security
- Access tokens disimpan di httpOnly cookie (SSR) atau memory (CSR)
- Refresh tokens di httpOnly cookie
- Tidak ada token di localStorage (XSS vulnerable)
- CSRF protection via SameSite cookie attribute

### Data Protection
- PIN di-hash dengan bcrypt (cost factor 12)
- OTP di-generate secara random (crypto-secure)
- Phone numbers di-sanitize dan diformat konsisten (+62xxx)
- PII (Personally Identifiable Information) dilindungi RLS

### WhatsApp API Security
- API token disimpan di environment variables (Edge Functions)
- Tidak pernah di-expose ke client
- Rate limiting di-enforce oleh WhatsApp Business API
- Template messages harus di-approve oleh Meta/WhatsApp
