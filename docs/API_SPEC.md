# API SPEC - Platform Digital Masjid

## Overview

Platform ini menggunakan Supabase sebagai backend, jadi mayoritas interaksi data dilakukan via **Supabase Client SDK** (bukan REST API tradisional). Edge Functions dipakai untuk logic yang butuh server-side processing. Realtime subscriptions untuk live updates.

---

## Supabase Client Queries per Feature

### Auth Queries

```typescript
// === SIGN IN WITH OTP ===
// Kirim OTP ke nomor WhatsApp
const { error } = await supabase.auth.signInWithOtp({
  phone: '+628123456789',
  options: {
    channel: 'whatsapp',
    data: { full_name: 'Ahmad', roles: ['orangtua'] },
  },
})

// === VERIFY OTP ===
const { data, error } = await supabase.auth.verifyOtp({
  phone: '+628123456789',
  token: '123456',
  type: 'sms',
})

// === GET CURRENT USER ===
const { data: { user } } = await supabase.auth.getUser()

// === GET SESSION ===
const { data: { session } } = await supabase.auth.getSession()

// === SIGN OUT ===
const { error } = await supabase.auth.signOut()

// === ADMIN: CREATE USER (service role only) ===
const { data, error } = await supabase.auth.admin.createUser({
  phone: '+628123456789',
  phone_confirm: true,
  user_metadata: { full_name: 'Ahmad', roles: ['orangtua'] },
})
```

### Profile Queries

```typescript
// === GET PROFILE ===
const { data: profile } = await supabase
  .from('profiles')
  .select('*')
  .eq('id', userId)
  .single()

// === UPDATE PROFILE ===
const { error } = await supabase
  .from('profiles')
  .update({
    full_name: 'Ahmad Fauzi',
    address: 'Jl. Masjid No. 1',
    avatar_url: 'https://...',
  })
  .eq('id', userId)

// === UPDATE PIN ===
const { error } = await supabase
  .from('profiles')
  .update({ pin_hash: hashedPin })
  .eq('id', userId)

// === LIST ALL USERS (admin) ===
const { data: users } = await supabase
  .from('profiles')
  .select('*, parent_children(child_id, children(full_name))')
  .order('created_at', { ascending: false })
```

### Children (Santri) Queries

```typescript
// === LIST CHILDREN (admin: all, parent: own) ===
const { data: children } = await supabase
  .from('children')
  .select(`
    *,
    parent_children(parent_id, profiles(full_name, phone)),
    class_enrollments(class_id, classes(name, programs(name)))
  `)
  .eq('is_active', true)
  .order('full_name')

// === GET CHILD DETAIL ===
const { data: child } = await supabase
  .from('children')
  .select(`
    *,
    parent_children(parent_id, relation, is_primary, profiles(full_name, phone)),
    class_enrollments(classes(id, name, programs(name, category)))
  `)
  .eq('id', childId)
  .single()

// === CREATE CHILD (admin) ===
const { data: child } = await supabase
  .from('children')
  .insert({
    full_name: 'Ahmad Junior',
    nickname: 'Junior',
    date_of_birth: '2018-05-15',
    gender: 'laki-laki',
    school_name: 'SDN 1',
    school_grade: 'Kelas 3',
  })
  .select()
  .single()

// === LINK PARENT TO CHILD ===
const { error } = await supabase
  .from('parent_children')
  .insert({
    parent_id: parentUserId,
    child_id: childId,
    relation: 'orangtua',
    is_primary: true,
  })

// === GET PARENT'S CHILDREN ===
const { data } = await supabase
  .from('parent_children')
  .select('children(*)')
  .eq('parent_id', userId)
```

### Attendance Queries

```typescript
// === RECORD ATTENDANCE (batch, pengajar) ===
const { error } = await supabase
  .from('attendance')
  .upsert(
    attendanceData.map((item) => ({
      session_id: sessionId,
      child_id: item.childId,
      status: item.status,
      notes: item.notes,
      leave_id: item.leaveId,
      recorded_by: teacherId,
    })),
    { onConflict: 'session_id,child_id' }
  )

// === GET ATTENDANCE FOR SESSION ===
const { data } = await supabase
  .from('attendance')
  .select(`
    *,
    children(full_name, nickname),
    leave_requests(id, reason, notes, status)
  `)
  .eq('session_id', sessionId)

// === GET CHILD ATTENDANCE (monthly) ===
const { data } = await supabase
  .from('attendance')
  .select(`
    *,
    sessions(date, start_time, classes(name, programs(name)))
  `)
  .eq('child_id', childId)
  .gte('created_at', startOfMonth)
  .lte('created_at', endOfMonth)
  .order('created_at', { ascending: false })

// === ATTENDANCE STATS ===
const { data, count } = await supabase
  .from('attendance')
  .select('status', { count: 'exact' })
  .eq('child_id', childId)
  .gte('created_at', startOfMonth)
```

### Leave Request Queries

```typescript
// === CREATE LEAVE REQUEST (orangtua) ===
const { data, error } = await supabase
  .from('leave_requests')
  .insert({
    child_id: childId,
    parent_id: userId,
    date: '2026-07-25',
    reason: 'izin',
    notes: 'Kontrol ke dokter gigi',
  })
  .select()
  .single()

// === APPROVE/REJECT LEAVE (pengajar/admin) ===
const { error } = await supabase
  .from('leave_requests')
  .update({
    status: 'approved', // atau 'rejected'
    reviewed_by: reviewerId,
    reviewed_at: new Date().toISOString(),
  })
  .eq('id', leaveId)

// === GET LEAVE REQUESTS FOR DATE (pengajar) ===
const { data } = await supabase
  .from('leave_requests')
  .select('*, children(full_name), profiles!parent_id(full_name)')
  .eq('date', targetDate)
  .eq('status', 'pending')
```

### Quran Progress Queries

```typescript
// === ADD QURAN PROGRESS (pengajar) ===
const { data, error } = await supabase
  .from('quran_progress')
  .insert({
    session_id: sessionId,
    child_id: childId,
    teacher_id: teacherId,
    program_type: 'iqra',
    iqra_volume: 3,
    iqra_page: 15,
    grade: 'jayyid_jiddan',
    notes: 'Bacaan sudah lancar, lanjut halaman berikutnya',
    date: new Date().toISOString().split('T')[0],
  })
  .select()
  .single()

// === GET LATEST PROGRESS PER PROGRAM ===
const { data } = await supabase
  .from('quran_progress')
  .select('*')
  .eq('child_id', childId)
  .order('date', { ascending: false })
  .limit(10)

// === GET PROGRESS HISTORY FOR PROGRAM TYPE ===
const { data } = await supabase
  .from('quran_progress')
  .select('*, profiles!teacher_id(full_name)')
  .eq('child_id', childId)
  .eq('program_type', 'hafalan')
  .order('date', { ascending: false })
```

### Study Progress Queries

```typescript
// === ADD STUDY PROGRESS (pengajar) ===
const { error } = await supabase
  .from('study_progress')
  .insert({
    session_id: sessionId,
    child_id: childId,
    teacher_id: teacherId,
    subject_id: subjectId,
    topic: 'Perkalian 2 digit',
    score: 85,
    grade: 'A',
    notes: 'Paham konsep, perlu latihan soal cerita',
    date: new Date().toISOString().split('T')[0],
  })

// === GET PROGRESS BY SUBJECT ===
const { data } = await supabase
  .from('study_progress')
  .select('*, subjects(name), profiles!teacher_id(full_name)')
  .eq('child_id', childId)
  .eq('subject_id', subjectId)
  .order('date', { ascending: false })
```

### Finance Queries

```typescript
// === GENERATE MONTHLY INVOICES (admin) ===
const { error } = await supabase
  .from('fee_invoices')
  .insert(
    activeChildren.map((child) => ({
      child_id: child.id,
      fee_type_id: sppFeeTypeId,
      amount: feeType.amount,
      period_month: 7,
      period_year: 2026,
      due_date: '2026-07-15',
      status: 'pending',
    }))
  )

// === GET INVOICES FOR CHILD (orangtua) ===
const { data } = await supabase
  .from('fee_invoices')
  .select('*, fee_types(name, frequency), profiles!verified_by(full_name)')
  .eq('child_id', childId)
  .order('period_year', { ascending: false })
  .order('period_month', { ascending: false })

// === MARK INVOICE AS PAID (admin) ===
const { error } = await supabase
  .from('fee_invoices')
  .update({
    status: 'paid',
    paid_at: new Date().toISOString(),
    paid_amount: amount,
    payment_method: 'transfer',
    receipt_url: receiptUrl,
    verified_by: adminId,
  })
  .eq('id', invoiceId)

// === CREATE DONATION ===
const { data, error } = await supabase
  .from('donations')
  .insert({
    donor_id: userId, // null for anonymous
    donor_name: 'Hamba Allah',
    amount: 500000,
    category: 'infaq',
    project_id: projectId, // optional
    is_anonymous: true,
    payment_method: 'transfer',
  })
  .select()
  .single()

// === GET FINANCIAL SUMMARY (public) ===
// Total donations this month
const { data: donations } = await supabase
  .from('donations')
  .select('amount')
  .eq('is_verified', true)
  .gte('created_at', startOfMonth)

// Total expenses this month
const { data: expenses } = await supabase
  .from('expenses')
  .select('amount')
  .gte('date', startOfMonth)

// === RECORD EXPENSE (admin) ===
const { error } = await supabase
  .from('expenses')
  .insert({
    category: 'operasional',
    description: 'Pembayaran listrik bulan Juli',
    amount: 1500000,
    date: '2026-07-10',
    receipt_url: 'https://...',
    recorded_by: adminId,
  })
```

### Program & Class Queries

```typescript
// === LIST PROGRAMS (public) ===
const { data: programs } = await supabase
  .from('programs')
  .select('*')
  .eq('is_active', true)
  .eq('is_public', true)
  .order('sort_order')

// === GET PROGRAM DETAIL ===
const { data: program } = await supabase
  .from('programs')
  .select(`
    *,
    classes(id, name, teacher_id, profiles!teacher_id(full_name),
            class_enrollments(count)),
    subjects(id, name)
  `)
  .eq('slug', slug)
  .single()

// === CREATE SESSION (pengajar) ===
const { data: session } = await supabase
  .from('sessions')
  .insert({
    class_id: classId,
    schedule_id: scheduleId,
    teacher_id: teacherId,
    date: new Date().toISOString().split('T')[0],
    start_time: '15:30',
    topic: 'Iqra Jilid 3 Halaman 15-20',
    status: 'ongoing',
  })
  .select()
  .single()

// === GET TODAY'S SESSIONS (pengajar) ===
const { data } = await supabase
  .from('sessions')
  .select(`
    *,
    classes(name, programs(name, category)),
    profiles!teacher_id(full_name)
  `)
  .eq('date', today)
  .eq('teacher_id', teacherId)
  .order('start_time')
```

### Project Queries

```typescript
// === LIST PROJECTS (public) ===
const { data: projects } = await supabase
  .from('projects')
  .select(`
    *,
    project_milestones(id, title, status),
    project_updates(id, title, created_at)
  `)
  .eq('is_public', true)
  .order('sort_order')

// === GET PROJECT DETAIL ===
const { data: project } = await supabase
  .from('projects')
  .select(`
    *,
    project_milestones(*, order: sort_order.asc),
    project_updates(
      *,
      profiles!posted_by(full_name),
      project_update_photos(*)
    )
  `)
  .eq('slug', slug)
  .single()

// === POST PROJECT UPDATE (admin) ===
const { data: update } = await supabase
  .from('project_updates')
  .insert({
    project_id: projectId,
    title: 'Progres Minggu ke-5',
    content: 'Pembangunan pondasi sudah selesai 80%...',
    posted_by: adminId,
  })
  .select()
  .single()

// Upload photos for update
const { error } = await supabase
  .from('project_update_photos')
  .insert(
    photos.map((photo, i) => ({
      update_id: update.id,
      url: photo.url,
      caption: photo.caption,
      sort_order: i,
    }))
  )
```

### Content Queries

```typescript
// === LIST ARTICLES (public) ===
const { data: articles } = await supabase
  .from('articles')
  .select('id, title, slug, excerpt, cover_image_url, tags, published_at')
  .eq('status', 'published')
  .order('published_at', { ascending: false })
  .range(offset, offset + limit - 1)

// === GET ARTICLE BY SLUG ===
const { data: article } = await supabase
  .from('articles')
  .select('*, profiles!author_id(full_name)')
  .eq('slug', slug)
  .eq('status', 'published')
  .single()

// === LIST ANNOUNCEMENTS ===
const { data: announcements } = await supabase
  .from('announcements')
  .select('*')
  .lte('published_at', new Date().toISOString())
  .or(`expires_at.is.null,expires_at.gte.${new Date().toISOString()}`)
  .order('is_pinned', { ascending: false })
  .order('published_at', { ascending: false })

// === GET PAGE CONTENT (CMS) ===
const { data: page } = await supabase
  .from('pages')
  .select('*')
  .eq('slug', 'home')
  .single()
```

### Notification Queries

```typescript
// === GET USER NOTIFICATIONS ===
const { data: notifications } = await supabase
  .from('notifications')
  .select('*')
  .eq('user_id', userId)
  .order('created_at', { ascending: false })
  .range(0, 19) // latest 20

// === MARK AS READ ===
const { error } = await supabase
  .from('notifications')
  .update({ is_read: true, read_at: new Date().toISOString() })
  .eq('id', notificationId)
  .eq('user_id', userId)

// === MARK ALL AS READ ===
const { error } = await supabase
  .from('notifications')
  .update({ is_read: true, read_at: new Date().toISOString() })
  .eq('user_id', userId)
  .eq('is_read', false)

// === UNREAD COUNT ===
const { count } = await supabase
  .from('notifications')
  .select('*', { count: 'exact', head: true })
  .eq('user_id', userId)
  .eq('is_read', false)
```

---

## Edge Functions

### 1. `send-wa-otp`

Mengirim OTP via WhatsApp Business API.

| Property | Value |
|----------|-------|
| **Path** | `/functions/v1/send-wa-otp` |
| **Method** | POST |
| **Auth** | Service role or authenticated |
| **Rate Limit** | 3 per 15 min per phone |

**Request Body:**
```json
{
  "phone": "+628123456789",
  "otp": "123456",
  "name": "Ahmad"
}
```

**Response:**
```json
{
  "success": true,
  "message_id": "wamid.xxx"
}
```

**Implementation:** Calls WhatsApp Business API with pre-approved OTP template.

### 2. `generate-credentials`

Admin membuat akun orang tua baru + generate temporary PIN.

| Property | Value |
|----------|-------|
| **Path** | `/functions/v1/generate-credentials` |
| **Method** | POST |
| **Auth** | Authenticated (admin only) |

**Request Body:**
```json
{
  "childData": {
    "full_name": "Ahmad Junior",
    "gender": "laki-laki",
    "date_of_birth": "2018-05-15",
    "school_name": "SDN 1",
    "school_grade": "Kelas 3"
  },
  "parentData": {
    "name": "Ahmad Senior",
    "phone": "+628123456789",
    "relation": "orangtua"
  },
  "classId": "uuid-of-class",
  "sendWhatsApp": true
}
```

**Response:**
```json
{
  "success": true,
  "child": { "id": "uuid", "full_name": "Ahmad Junior" },
  "parent": { "id": "uuid", "full_name": "Ahmad Senior" },
  "tempPin": "847291",
  "whatsappSent": true
}
```

**Process:**
1. Generate random 6-digit PIN
2. Create auth user via `supabase.auth.admin.createUser`
3. Hash PIN and store in profile
4. Create child record
5. Create parent_children link
6. Enroll child in class (if classId provided)
7. Create invitation record
8. Send credentials via WhatsApp (if sendWhatsApp=true)

### 3. `notify`

Kirim notifikasi ke user (in-app + optional WhatsApp).

| Property | Value |
|----------|-------|
| **Path** | `/functions/v1/notify` |
| **Method** | POST |
| **Auth** | Service role or authenticated (admin) |

**Request Body:**
```json
{
  "userId": "uuid-of-user",
  "type": "attendance",
  "title": "Ahmad tidak hadir",
  "body": "Ahmad tidak hadir pada sesi Ngaji Iqra hari ini.",
  "data": {
    "child_id": "uuid",
    "session_id": "uuid",
    "action_url": "/anak/uuid"
  },
  "channels": ["in_app", "whatsapp"]
}
```

**Response:**
```json
{
  "success": true,
  "notification_id": "uuid",
  "whatsapp_sent": true
}
```

**Process:**
1. Insert notification record in database
2. If "whatsapp" channel: send WA message via API
3. If "push" channel (future): send web push notification

### Batch Notification (untuk attendance)

```json
{
  "type": "attendance_batch",
  "session_id": "uuid",
  "absent_children": [
    { "child_id": "uuid", "parent_ids": ["uuid1", "uuid2"] }
  ]
}
```

Ini akan mengirim notifikasi ke semua orang tua dari anak yang tidak hadir.

---

## Supabase Realtime Subscriptions

### Attendance Updates (Jamaah App)

Orang tua mendapat live update ketika attendance anak direcord:

```typescript
// apps/jamaah - realtime attendance for parent
const channel = supabase
  .channel('attendance-updates')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'attendance',
      filter: `child_id=in.(${childIds.join(',')})`,
    },
    (payload) => {
      // Update UI with new attendance record
      queryClient.invalidateQueries(['attendance', payload.new.child_id])
      toast({
        title: 'Kehadiran Tercatat',
        description: `${childName} - ${payload.new.status}`,
      })
    }
  )
  .subscribe()
```

### Notification Updates (Jamaah App)

Real-time notifications:

```typescript
// apps/jamaah - realtime notifications
const channel = supabase
  .channel('notifications')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'notifications',
      filter: `user_id=eq.${userId}`,
    },
    (payload) => {
      // Show toast / update badge count
      setUnreadCount((prev) => prev + 1)
      toast({
        title: payload.new.title,
        description: payload.new.body,
      })
    }
  )
  .subscribe()
```

### Progress Updates (Jamaah App)

Orang tua mendapat update ketika progress ngaji anak di-input:

```typescript
// apps/jamaah - realtime quran progress
const channel = supabase
  .channel('quran-progress')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'quran_progress',
      filter: `child_id=in.(${childIds.join(',')})`,
    },
    (payload) => {
      queryClient.invalidateQueries(['quran-progress', payload.new.child_id])
    }
  )
  .subscribe()
```

### Session Updates (Admin App)

Pengajar mendapat update ketika ada leave request baru:

```typescript
// apps/admin - realtime leave requests
const channel = supabase
  .channel('leave-requests')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'leave_requests',
      filter: `status=eq.pending`,
    },
    (payload) => {
      // Notify teacher about new leave request
      queryClient.invalidateQueries(['leave-requests'])
    }
  )
  .subscribe()
```

### Donation Updates (Landing/Jamaah)

Live update ketika ada donasi baru terverifikasi:

```typescript
// apps/landing - realtime donation count
const channel = supabase
  .channel('donations')
  .on(
    'postgres_changes',
    {
      event: 'UPDATE',
      schema: 'public',
      table: 'donations',
      filter: `is_verified=eq.true`,
    },
    (payload) => {
      // Update donation total on keuangan page
      queryClient.invalidateQueries(['donation-total'])
    }
  )
  .subscribe()
```

---

## Webhook Endpoints (Future)

### Payment Gateway Webhook

Untuk integrasi payment gateway di masa depan (Midtrans, Xendit, dll):

```typescript
// apps/jamaah/app/api/webhooks/payment/route.ts
// atau apps/admin/app/api/webhooks/payment/route.ts

export async function POST(req: Request) {
  const payload = await req.json()

  // Verify webhook signature
  const signature = req.headers.get('x-callback-token')
  if (!verifySignature(signature, payload)) {
    return new Response('Unauthorized', { status: 401 })
  }

  const { order_id, transaction_status, gross_amount } = payload

  if (transaction_status === 'settlement') {
    // Parse order_id to get invoice/donation info
    // Format: "INV-{invoiceId}" or "DON-{donationId}"

    if (order_id.startsWith('INV-')) {
      const invoiceId = order_id.replace('INV-', '')
      await supabase
        .from('fee_invoices')
        .update({
          status: 'paid',
          paid_at: new Date().toISOString(),
          paid_amount: parseInt(gross_amount),
          payment_method: 'gateway',
        })
        .eq('id', invoiceId)
    }

    if (order_id.startsWith('DON-')) {
      const donationId = order_id.replace('DON-', '')
      await supabase
        .from('donations')
        .update({
          is_verified: true,
          verified_at: new Date().toISOString(),
        })
        .eq('id', donationId)
    }

    // Send notification to user
    await supabase.functions.invoke('notify', {
      body: {
        userId: getUserFromOrder(order_id),
        type: 'payment',
        title: 'Pembayaran Berhasil',
        body: `Pembayaran sebesar ${formatRupiah(gross_amount)} telah diterima.`,
      },
    })
  }

  return new Response('OK', { status: 200 })
}
```

### On-Demand ISR Revalidation

```typescript
// apps/landing/app/api/revalidate/route.ts

export async function POST(req: Request) {
  const { secret, path } = await req.json()

  if (secret !== process.env.REVALIDATE_SECRET) {
    return new Response('Unauthorized', { status: 401 })
  }

  try {
    revalidatePath(path)
    return Response.json({ revalidated: true, path })
  } catch (err) {
    return Response.json({ revalidated: false, error: err.message }, { status: 500 })
  }
}

// Called from admin after content update:
// POST /api/revalidate { secret: "xxx", path: "/program" }
// POST /api/revalidate { secret: "xxx", path: "/artikel/slug-artikel" }
```

---

## Storage Buckets

### Bucket Configuration

| Bucket | Access | Max File Size | Allowed MIME Types | Usage |
|--------|--------|--------------|-------------------|-------|
| `avatars` | Public | 2MB | image/jpeg, image/png, image/webp | Foto profil user |
| `gallery` | Public | 5MB | image/jpeg, image/png, image/webp | Galeri foto masjid |
| `projects` | Public | 5MB | image/jpeg, image/png, image/webp | Foto update proyek |
| `receipts` | Private | 5MB | image/jpeg, image/png, image/pdf | Bukti bayar SPP |
| `documents` | Private | 10MB | application/pdf, image/* | Dokumen rapor, dll |
| `articles` | Public | 5MB | image/jpeg, image/png, image/webp | Cover image artikel |

### Upload Example

```typescript
// Upload avatar
async function uploadAvatar(file: File, userId: string) {
  const fileExt = file.name.split('.').pop()
  const filePath = `${userId}/avatar.${fileExt}`

  const { data, error } = await supabase.storage
    .from('avatars')
    .upload(filePath, file, {
      cacheControl: '3600',
      upsert: true, // Replace if exists
    })

  if (error) throw error

  // Get public URL
  const { data: { publicUrl } } = supabase.storage
    .from('avatars')
    .getPublicUrl(filePath)

  // Update profile
  await supabase
    .from('profiles')
    .update({ avatar_url: publicUrl })
    .eq('id', userId)

  return publicUrl
}

// Upload project photo
async function uploadProjectPhoto(file: File, projectId: string) {
  const fileName = `${projectId}/${Date.now()}-${file.name}`

  const { data, error } = await supabase.storage
    .from('projects')
    .upload(fileName, file, { cacheControl: '86400' })

  const { data: { publicUrl } } = supabase.storage
    .from('projects')
    .getPublicUrl(fileName)

  return publicUrl
}

// Upload receipt (private)
async function uploadReceipt(file: File, invoiceId: string) {
  const fileName = `${invoiceId}/${Date.now()}-${file.name}`

  const { data, error } = await supabase.storage
    .from('receipts')
    .upload(fileName, file)

  // Private bucket - generate signed URL for viewing
  const { data: signedUrl } = await supabase.storage
    .from('receipts')
    .createSignedUrl(fileName, 3600) // 1 hour expiry

  return signedUrl
}
```

### Storage RLS Policies

```sql
-- avatars bucket: user can upload own avatar
CREATE POLICY "avatar_upload" ON storage.objects
  FOR INSERT TO authenticated
  WITH CHECK (
    bucket_id = 'avatars' AND
    (storage.foldername(name))[1] = auth.uid()::text
  );

-- avatars bucket: public read
CREATE POLICY "avatar_read" ON storage.objects
  FOR SELECT TO anon
  USING (bucket_id = 'avatars');

-- gallery/projects: admin upload
CREATE POLICY "gallery_upload" ON storage.objects
  FOR INSERT TO authenticated
  WITH CHECK (
    bucket_id IN ('gallery', 'projects', 'articles') AND
    is_admin()
  );

-- gallery/projects: public read
CREATE POLICY "gallery_read" ON storage.objects
  FOR SELECT TO anon
  USING (bucket_id IN ('gallery', 'projects', 'articles'));

-- receipts: user uploads own, admin reads all
CREATE POLICY "receipt_upload" ON storage.objects
  FOR INSERT TO authenticated
  WITH CHECK (bucket_id = 'receipts');

CREATE POLICY "receipt_read" ON storage.objects
  FOR SELECT TO authenticated
  USING (
    bucket_id = 'receipts' AND
    (is_admin() OR (storage.foldername(name))[1] IN (
      SELECT id::text FROM fee_invoices
      WHERE child_id IN (
        SELECT child_id FROM parent_children WHERE parent_id = auth.uid()
      )
    ))
  );
```

---

## Error Handling

### Standard Error Response Format

```typescript
interface ApiError {
  code: string           // e.g., 'AUTH_INVALID_OTP'
  message: string        // User-friendly message in Indonesian
  details?: any          // Additional context (dev only)
}

// Error codes
const ERROR_CODES = {
  AUTH_PHONE_NOT_FOUND: 'Nomor WhatsApp tidak terdaftar',
  AUTH_INVALID_OTP: 'Kode OTP salah atau sudah expired',
  AUTH_INVALID_PIN: 'PIN salah',
  AUTH_RATE_LIMITED: 'Terlalu banyak percobaan, coba lagi nanti',
  AUTH_ACCOUNT_DISABLED: 'Akun dinonaktifkan, hubungi pengurus masjid',
  DATA_NOT_FOUND: 'Data tidak ditemukan',
  DATA_VALIDATION_ERROR: 'Data tidak valid',
  PERMISSION_DENIED: 'Anda tidak memiliki akses',
  SERVER_ERROR: 'Terjadi kesalahan, silakan coba lagi',
}
```

### Client-Side Error Handling

```typescript
// packages/db/src/queries/utils.ts

export async function queryWithError<T>(
  queryFn: () => Promise<{ data: T | null; error: any }>
): Promise<T> {
  const { data, error } = await queryFn()

  if (error) {
    if (error.code === 'PGRST116') throw new Error(ERROR_CODES.DATA_NOT_FOUND)
    if (error.code === '42501') throw new Error(ERROR_CODES.PERMISSION_DENIED)
    throw new Error(error.message || ERROR_CODES.SERVER_ERROR)
  }

  if (!data) throw new Error(ERROR_CODES.DATA_NOT_FOUND)

  return data
}
```
