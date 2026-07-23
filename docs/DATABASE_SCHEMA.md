# DATABASE SCHEMA - Platform Digital Masjid

## Overview

Database menggunakan PostgreSQL via Supabase. Semua tabel dilindungi Row Level Security (RLS). Schema ini mencakup seluruh tabel yang dibutuhkan untuk platform digital masjid.

---

## Enums

```sql
-- Role enum
CREATE TYPE user_role AS ENUM (
  'superadmin',
  'pengurus',
  'pengajar',
  'orangtua',
  'donatur'
);

-- Gender enum
CREATE TYPE gender AS ENUM ('laki-laki', 'perempuan');

-- Attendance status
CREATE TYPE attendance_status AS ENUM ('hadir', 'izin', 'sakit', 'alfa');

-- Leave request status
CREATE TYPE leave_status AS ENUM ('pending', 'approved', 'rejected');

-- Quran program type
CREATE TYPE quran_program AS ENUM ('iqra', 'al-quran', 'hafalan', 'doa', 'hadits');

-- Hafalan status
CREATE TYPE hafalan_status AS ENUM ('belum', 'proses', 'hafal');

-- Invoice status
CREATE TYPE invoice_status AS ENUM ('pending', 'paid', 'overdue', 'cancelled');

-- Donation category
CREATE TYPE donation_category AS ENUM ('infaq', 'sedekah', 'zakat', 'proyek', 'umum');

-- Project status
CREATE TYPE project_status AS ENUM ('planning', 'active', 'completed', 'on-hold');

-- Milestone status
CREATE TYPE milestone_status AS ENUM ('pending', 'in-progress', 'completed');

-- Notification type
CREATE TYPE notification_type AS ENUM (
  'attendance',
  'progress',
  'payment',
  'announcement',
  'project',
  'general'
);

-- Article status
CREATE TYPE article_status AS ENUM ('draft', 'published', 'archived');

-- Invitation status
CREATE TYPE invitation_status AS ENUM ('pending', 'accepted', 'expired');

-- Day of week
CREATE TYPE day_of_week AS ENUM (
  'senin', 'selasa', 'rabu', 'kamis', 'jumat', 'sabtu', 'ahad'
);
```

---

## Tables

### 1. profiles

Extends Supabase auth.users dengan data profil tambahan.

```sql
CREATE TABLE profiles (
  id            UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name     TEXT NOT NULL,
  phone         TEXT UNIQUE NOT NULL,
  pin_hash      TEXT,                          -- bcrypt hashed 6-digit PIN
  roles         user_role[] NOT NULL DEFAULT ARRAY['orangtua']::user_role[],
  avatar_url    TEXT,
  address       TEXT,
  is_active     BOOLEAN NOT NULL DEFAULT TRUE,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_profiles_phone ON profiles(phone);
CREATE INDEX idx_profiles_roles ON profiles USING GIN(roles);
CREATE INDEX idx_profiles_is_active ON profiles(is_active);
```

### 2. children

Data anak/santri.

```sql
CREATE TABLE children (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  full_name       TEXT NOT NULL,
  nickname        TEXT,
  date_of_birth   DATE,
  gender          gender NOT NULL,
  photo_url       TEXT,
  school_name     TEXT,
  school_grade    TEXT,                        -- e.g., "Kelas 3 SD"
  notes           TEXT,
  is_active       BOOLEAN NOT NULL DEFAULT TRUE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_children_is_active ON children(is_active);
```

### 3. parent_children

Relasi many-to-many antara orang tua dan anak.

```sql
CREATE TABLE parent_children (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parent_id   UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  child_id    UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  relation    TEXT NOT NULL DEFAULT 'orangtua',  -- orangtua, wali, kakek/nenek
  is_primary  BOOLEAN NOT NULL DEFAULT FALSE,    -- primary guardian
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  UNIQUE(parent_id, child_id)
);

-- Indexes
CREATE INDEX idx_parent_children_parent ON parent_children(parent_id);
CREATE INDEX idx_parent_children_child ON parent_children(child_id);
```

### 4. programs

Program yang tersedia di masjid (ngaji, belajar kelompok, dll).

```sql
CREATE TABLE programs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            TEXT NOT NULL,
  slug            TEXT UNIQUE NOT NULL,
  description     TEXT,
  category        TEXT NOT NULL,               -- 'ngaji', 'belajar_kelompok'
  image_url       TEXT,
  is_active       BOOLEAN NOT NULL DEFAULT TRUE,
  is_public       BOOLEAN NOT NULL DEFAULT TRUE, -- tampil di landing
  sort_order      INT NOT NULL DEFAULT 0,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_programs_slug ON programs(slug);
CREATE INDEX idx_programs_category ON programs(category);
CREATE INDEX idx_programs_is_public ON programs(is_public) WHERE is_public = TRUE;
```

### 5. subjects

Mata pelajaran untuk program belajar kelompok.

```sql
CREATE TABLE subjects (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  program_id  UUID NOT NULL REFERENCES programs(id) ON DELETE CASCADE,
  name        TEXT NOT NULL,                   -- 'Matematika', 'IPA', 'Bahasa Indonesia'
  description TEXT,
  is_active   BOOLEAN NOT NULL DEFAULT TRUE,
  sort_order  INT NOT NULL DEFAULT 0,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_subjects_program ON subjects(program_id);
```

### 6. classes

Kelas/kelompok belajar.

```sql
CREATE TABLE classes (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  program_id  UUID NOT NULL REFERENCES programs(id) ON DELETE CASCADE,
  name        TEXT NOT NULL,                   -- 'Iqra Jilid 1', 'Kelas Hafalan Juz 30'
  description TEXT,
  teacher_id  UUID REFERENCES profiles(id),   -- pengajar utama
  max_students INT,
  is_active   BOOLEAN NOT NULL DEFAULT TRUE,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_classes_program ON classes(program_id);
CREATE INDEX idx_classes_teacher ON classes(teacher_id);
```

### 7. class_enrollments

Santri yang terdaftar di kelas.

```sql
CREATE TABLE class_enrollments (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id    UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
  child_id    UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  enrolled_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  is_active   BOOLEAN NOT NULL DEFAULT TRUE,

  UNIQUE(class_id, child_id)
);

-- Indexes
CREATE INDEX idx_enrollments_class ON class_enrollments(class_id);
CREATE INDEX idx_enrollments_child ON class_enrollments(child_id);
```

### 8. schedules

Jadwal mingguan per kelas.

```sql
CREATE TABLE schedules (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id    UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
  day         day_of_week NOT NULL,
  start_time  TIME NOT NULL,
  end_time    TIME NOT NULL,
  location    TEXT,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_schedules_class ON schedules(class_id);
CREATE INDEX idx_schedules_day ON schedules(day);
```

### 9. sessions

Sesi/pertemuan aktual (instance dari schedule).

```sql
CREATE TABLE sessions (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  class_id    UUID NOT NULL REFERENCES classes(id) ON DELETE CASCADE,
  schedule_id UUID REFERENCES schedules(id),
  teacher_id  UUID NOT NULL REFERENCES profiles(id),
  date        DATE NOT NULL,
  start_time  TIME NOT NULL,
  end_time    TIME,
  topic       TEXT,
  notes       TEXT,
  status      TEXT NOT NULL DEFAULT 'scheduled', -- scheduled, ongoing, completed, cancelled
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_sessions_class ON sessions(class_id);
CREATE INDEX idx_sessions_teacher ON sessions(teacher_id);
CREATE INDEX idx_sessions_date ON sessions(date);
CREATE INDEX idx_sessions_class_date ON sessions(class_id, date);
```

### 10. leave_requests

Pengajuan izin dari orang tua. (Defined before attendance because attendance references it.)

```sql
CREATE TABLE leave_requests (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  child_id    UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  parent_id   UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  date        DATE NOT NULL,
  reason      attendance_status NOT NULL,      -- izin atau sakit
  notes       TEXT,
  status      leave_status NOT NULL DEFAULT 'pending',
  reviewed_by UUID REFERENCES profiles(id),
  reviewed_at TIMESTAMPTZ,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_leave_child ON leave_requests(child_id);
CREATE INDEX idx_leave_parent ON leave_requests(parent_id);
CREATE INDEX idx_leave_date ON leave_requests(date);
CREATE INDEX idx_leave_status ON leave_requests(status);
```

### 11. attendance

Kehadiran santri per sesi.

```sql
CREATE TABLE attendance (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id  UUID NOT NULL REFERENCES sessions(id) ON DELETE CASCADE,
  child_id    UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  status      attendance_status NOT NULL DEFAULT 'hadir',
  leave_id    UUID REFERENCES leave_requests(id),  -- link ke izin ortu
  notes       TEXT,
  recorded_by UUID NOT NULL REFERENCES profiles(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  UNIQUE(session_id, child_id)
);

-- Indexes
CREATE INDEX idx_attendance_session ON attendance(session_id);
CREATE INDEX idx_attendance_child ON attendance(child_id);
CREATE INDEX idx_attendance_status ON attendance(status);
CREATE INDEX idx_attendance_child_date ON attendance(child_id, created_at);
```

### 12. quran_progress

Progress ngaji santri (Iqra, Al-Quran, Hafalan, Doa, Hadits).

```sql
CREATE TABLE quran_progress (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id      UUID REFERENCES sessions(id) ON DELETE SET NULL,
  child_id        UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  teacher_id      UUID NOT NULL REFERENCES profiles(id),
  program_type    quran_program NOT NULL,
  date            DATE NOT NULL DEFAULT CURRENT_DATE,

  -- Iqra specific
  iqra_volume     INT,                         -- Jilid 1-6
  iqra_page       INT,                         -- Halaman

  -- Al-Quran specific
  surah_number    INT,                         -- 1-114
  surah_name      TEXT,
  ayat_from       INT,
  ayat_to         INT,
  juz             INT,                         -- 1-30

  -- Hafalan specific
  hafalan_surah   INT,
  hafalan_status  hafalan_status,

  -- Doa / Hadits specific
  item_name       TEXT,                        -- Nama doa / hadits
  item_status     hafalan_status,

  -- Common fields
  grade           TEXT,                        -- A, B, C, D / mumtaz, jayyid jiddan, dll
  notes           TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_quran_child ON quran_progress(child_id);
CREATE INDEX idx_quran_session ON quran_progress(session_id);
CREATE INDEX idx_quran_program ON quran_progress(program_type);
CREATE INDEX idx_quran_date ON quran_progress(date);
CREATE INDEX idx_quran_child_program ON quran_progress(child_id, program_type);
```

### 13. study_progress

Progress belajar kelompok (mata pelajaran umum).

```sql
CREATE TABLE study_progress (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id  UUID REFERENCES sessions(id) ON DELETE SET NULL,
  child_id    UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  teacher_id  UUID NOT NULL REFERENCES profiles(id),
  subject_id  UUID NOT NULL REFERENCES subjects(id),
  date        DATE NOT NULL DEFAULT CURRENT_DATE,
  topic       TEXT,                            -- Materi hari ini
  score       DECIMAL(5,2),                    -- Nilai 0-100
  grade       TEXT,                            -- A, B, C, D
  notes       TEXT,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_study_child ON study_progress(child_id);
CREATE INDEX idx_study_session ON study_progress(session_id);
CREATE INDEX idx_study_subject ON study_progress(subject_id);
CREATE INDEX idx_study_date ON study_progress(date);
CREATE INDEX idx_study_child_subject ON study_progress(child_id, subject_id);
```

### 14. progress_reports

Rapor digital per semester/periode.

```sql
CREATE TABLE progress_reports (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  child_id        UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  period_name     TEXT NOT NULL,               -- 'Semester 1 2024/2025'
  period_start    DATE NOT NULL,
  period_end      DATE NOT NULL,
  total_sessions  INT NOT NULL DEFAULT 0,
  attended        INT NOT NULL DEFAULT 0,
  quran_summary   JSONB,                       -- Ringkasan progress ngaji
  study_summary   JSONB,                       -- Ringkasan progress belajar
  teacher_notes   TEXT,
  generated_by    UUID NOT NULL REFERENCES profiles(id),
  is_published    BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_reports_child ON progress_reports(child_id);
CREATE INDEX idx_reports_period ON progress_reports(period_start, period_end);
```

### 15. fee_types

Jenis biaya (SPP bulanan, pendaftaran, dll).

```sql
CREATE TABLE fee_types (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name        TEXT NOT NULL,                   -- 'SPP Bulanan', 'Biaya Pendaftaran'
  amount      BIGINT NOT NULL,                 -- dalam Rupiah
  frequency   TEXT NOT NULL DEFAULT 'monthly', -- monthly, once, yearly
  description TEXT,
  is_active   BOOLEAN NOT NULL DEFAULT TRUE,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 16. fee_invoices

Tagihan per santri.

```sql
CREATE TABLE fee_invoices (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  child_id        UUID NOT NULL REFERENCES children(id) ON DELETE CASCADE,
  fee_type_id     UUID NOT NULL REFERENCES fee_types(id),
  amount          BIGINT NOT NULL,             -- nominal tagihan
  period_month    INT,                         -- 1-12 (untuk bulanan)
  period_year     INT,
  due_date        DATE,
  status          invoice_status NOT NULL DEFAULT 'pending',
  paid_at         TIMESTAMPTZ,
  paid_amount     BIGINT,
  payment_method  TEXT,                        -- 'transfer', 'tunai'
  receipt_url     TEXT,                        -- bukti bayar
  notes           TEXT,
  verified_by     UUID REFERENCES profiles(id),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_invoices_child ON fee_invoices(child_id);
CREATE INDEX idx_invoices_status ON fee_invoices(status);
CREATE INDEX idx_invoices_period ON fee_invoices(period_year, period_month);
CREATE INDEX idx_invoices_due ON fee_invoices(due_date) WHERE status = 'pending';
```

### 17. donations

Catatan donasi.

```sql
CREATE TABLE donations (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  donor_id        UUID REFERENCES profiles(id), -- NULL untuk donasi anonim
  donor_name      TEXT,                          -- nama donatur (bisa anonim)
  donor_phone     TEXT,
  amount          BIGINT NOT NULL,
  category        donation_category NOT NULL DEFAULT 'umum',
  project_id      UUID REFERENCES projects(id), -- link ke proyek (opsional)
  message         TEXT,
  is_anonymous    BOOLEAN NOT NULL DEFAULT FALSE,
  payment_method  TEXT,
  receipt_url     TEXT,
  is_verified     BOOLEAN NOT NULL DEFAULT FALSE,
  verified_by     UUID REFERENCES profiles(id),
  verified_at     TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_donations_donor ON donations(donor_id);
CREATE INDEX idx_donations_category ON donations(category);
CREATE INDEX idx_donations_project ON donations(project_id);
CREATE INDEX idx_donations_verified ON donations(is_verified);
CREATE INDEX idx_donations_created ON donations(created_at);
```

### 18. expenses

Pencatatan pengeluaran masjid.

```sql
CREATE TABLE expenses (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  category        TEXT NOT NULL,               -- 'operasional', 'pembangunan', 'kegiatan', dll
  description     TEXT NOT NULL,
  amount          BIGINT NOT NULL,
  date            DATE NOT NULL DEFAULT CURRENT_DATE,
  receipt_url     TEXT,
  project_id      UUID REFERENCES projects(id),
  recorded_by     UUID NOT NULL REFERENCES profiles(id),
  approved_by     UUID REFERENCES profiles(id),
  notes           TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_expenses_category ON expenses(category);
CREATE INDEX idx_expenses_date ON expenses(date);
CREATE INDEX idx_expenses_project ON expenses(project_id);
```

### 19. projects

Proyek pembangunan/kegiatan masjid.

```sql
CREATE TABLE projects (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title           TEXT NOT NULL,
  slug            TEXT UNIQUE NOT NULL,
  description     TEXT,
  cover_image_url TEXT,
  target_amount   BIGINT,                      -- target dana
  collected_amount BIGINT NOT NULL DEFAULT 0,  -- dana terkumpul (calculated)
  status          project_status NOT NULL DEFAULT 'planning',
  start_date      DATE,
  end_date        DATE,
  is_public       BOOLEAN NOT NULL DEFAULT TRUE,
  sort_order      INT NOT NULL DEFAULT 0,
  created_by      UUID NOT NULL REFERENCES profiles(id),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_projects_slug ON projects(slug);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_is_public ON projects(is_public) WHERE is_public = TRUE;
```

### 20. project_milestones

Milestone proyek.

```sql
CREATE TABLE project_milestones (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id  UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  title       TEXT NOT NULL,
  description TEXT,
  target_date DATE,
  status      milestone_status NOT NULL DEFAULT 'pending',
  sort_order  INT NOT NULL DEFAULT 0,
  completed_at TIMESTAMPTZ,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_milestones_project ON project_milestones(project_id);
CREATE INDEX idx_milestones_status ON project_milestones(status);
```

### 21. project_updates

Update/progress report proyek.

```sql
CREATE TABLE project_updates (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id  UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  title       TEXT NOT NULL,
  content     TEXT,
  posted_by   UUID NOT NULL REFERENCES profiles(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_updates_project ON project_updates(project_id);
CREATE INDEX idx_updates_created ON project_updates(created_at);
```

### 22. project_update_photos

Foto yang dilampirkan pada update proyek.

```sql
CREATE TABLE project_update_photos (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  update_id   UUID NOT NULL REFERENCES project_updates(id) ON DELETE CASCADE,
  url         TEXT NOT NULL,
  caption     TEXT,
  sort_order  INT NOT NULL DEFAULT 0,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_update_photos_update ON project_update_photos(update_id);
```

### 23. announcements

Pengumuman masjid.

```sql
CREATE TABLE announcements (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title       TEXT NOT NULL,
  content     TEXT NOT NULL,
  image_url   TEXT,
  is_pinned   BOOLEAN NOT NULL DEFAULT FALSE,
  is_public   BOOLEAN NOT NULL DEFAULT FALSE,  -- tampil di landing
  target_roles user_role[],                    -- NULL = semua
  published_at TIMESTAMPTZ,
  expires_at  TIMESTAMPTZ,
  created_by  UUID NOT NULL REFERENCES profiles(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_announcements_published ON announcements(published_at);
CREATE INDEX idx_announcements_pinned ON announcements(is_pinned) WHERE is_pinned = TRUE;
CREATE INDEX idx_announcements_public ON announcements(is_public) WHERE is_public = TRUE;
```

### 24. gallery

Galeri foto masjid.

```sql
CREATE TABLE gallery (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title       TEXT,
  description TEXT,
  url         TEXT NOT NULL,
  thumbnail_url TEXT,
  category    TEXT,                             -- 'kegiatan', 'fasilitas', 'acara'
  is_public   BOOLEAN NOT NULL DEFAULT TRUE,
  sort_order  INT NOT NULL DEFAULT 0,
  uploaded_by UUID NOT NULL REFERENCES profiles(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_gallery_category ON gallery(category);
CREATE INDEX idx_gallery_public ON gallery(is_public) WHERE is_public = TRUE;
```

### 25. notifications

Notifikasi in-app dan push.

```sql
CREATE TABLE notifications (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  type        notification_type NOT NULL,
  title       TEXT NOT NULL,
  body        TEXT,
  data        JSONB,                           -- additional context (child_id, session_id, etc)
  is_read     BOOLEAN NOT NULL DEFAULT FALSE,
  read_at     TIMESTAMPTZ,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, is_read) WHERE is_read = FALSE;
CREATE INDEX idx_notifications_created ON notifications(created_at);
CREATE INDEX idx_notifications_type ON notifications(type);
```

### 26. articles

Artikel/blog untuk landing page.

```sql
CREATE TABLE articles (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title           TEXT NOT NULL,
  slug            TEXT UNIQUE NOT NULL,
  excerpt         TEXT,
  content         TEXT NOT NULL,
  cover_image_url TEXT,
  author_id       UUID NOT NULL REFERENCES profiles(id),
  status          article_status NOT NULL DEFAULT 'draft',
  tags            TEXT[],
  meta_title      TEXT,                        -- SEO
  meta_description TEXT,                       -- SEO
  published_at    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_articles_slug ON articles(slug);
CREATE INDEX idx_articles_status ON articles(status);
CREATE INDEX idx_articles_published ON articles(published_at) WHERE status = 'published';
CREATE INDEX idx_articles_tags ON articles USING GIN(tags);
```

### 27. pages

Konten halaman landing yang editable (CMS-like).

```sql
CREATE TABLE pages (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug        TEXT UNIQUE NOT NULL,            -- 'home', 'tentang', 'hero'
  title       TEXT NOT NULL,
  content     JSONB NOT NULL,                  -- structured content blocks
  meta_title  TEXT,
  meta_description TEXT,
  updated_by  UUID REFERENCES profiles(id),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_pages_slug ON pages(slug);
```

### 28. invitations

Undangan/credentials yang di-generate admin untuk orang tua baru.

```sql
CREATE TABLE invitations (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone           TEXT NOT NULL,
  child_id        UUID REFERENCES children(id),
  generated_pin   TEXT NOT NULL,               -- temporary PIN (hashed)
  status          invitation_status NOT NULL DEFAULT 'pending',
  invited_by      UUID NOT NULL REFERENCES profiles(id),
  accepted_at     TIMESTAMPTZ,
  expires_at      TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '30 days'),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_invitations_phone ON invitations(phone);
CREATE INDEX idx_invitations_status ON invitations(status);
CREATE INDEX idx_invitations_child ON invitations(child_id);
```

---

## Row Level Security (RLS) Policies

### Helper Functions

```sql
-- Get current user's roles
CREATE OR REPLACE FUNCTION get_user_roles()
RETURNS user_role[] AS $$
  SELECT roles FROM profiles WHERE id = auth.uid();
$$ LANGUAGE sql SECURITY DEFINER STABLE;

-- Check if user has specific role
CREATE OR REPLACE FUNCTION has_role(required_role user_role)
RETURNS BOOLEAN AS $$
  SELECT required_role = ANY(get_user_roles());
$$ LANGUAGE sql SECURITY DEFINER STABLE;

-- Check if user is admin (superadmin or pengurus)
CREATE OR REPLACE FUNCTION is_admin()
RETURNS BOOLEAN AS $$
  SELECT 'superadmin' = ANY(get_user_roles()) OR 'pengurus' = ANY(get_user_roles());
$$ LANGUAGE sql SECURITY DEFINER STABLE;

-- Check if user is teacher
CREATE OR REPLACE FUNCTION is_teacher()
RETURNS BOOLEAN AS $$
  SELECT 'pengajar' = ANY(get_user_roles());
$$ LANGUAGE sql SECURITY DEFINER STABLE;

-- Check if user is parent of child
CREATE OR REPLACE FUNCTION is_parent_of(check_child_id UUID)
RETURNS BOOLEAN AS $$
  SELECT EXISTS (
    SELECT 1 FROM parent_children
    WHERE parent_id = auth.uid() AND child_id = check_child_id
  );
$$ LANGUAGE sql SECURITY DEFINER STABLE;
```

### RLS Policies per Table

```sql
-- ============================================
-- PROFILES
-- ============================================
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Semua authenticated user bisa lihat profil (untuk display nama, dll)
CREATE POLICY "profiles_select_authenticated"
  ON profiles FOR SELECT
  TO authenticated
  USING (TRUE);

-- User bisa update profil sendiri
CREATE POLICY "profiles_update_own"
  ON profiles FOR UPDATE
  TO authenticated
  USING (id = auth.uid())
  WITH CHECK (id = auth.uid());

-- Admin bisa update semua profil
CREATE POLICY "profiles_update_admin"
  ON profiles FOR UPDATE
  TO authenticated
  USING (is_admin());

-- Admin bisa insert profil baru (atau user insert own on signup)
CREATE POLICY "profiles_insert_admin"
  ON profiles FOR INSERT
  TO authenticated
  WITH CHECK (is_admin() OR id = auth.uid());

-- ============================================
-- CHILDREN
-- ============================================
ALTER TABLE children ENABLE ROW LEVEL SECURITY;

-- Admin & pengajar bisa lihat semua anak
CREATE POLICY "children_select_staff"
  ON children FOR SELECT
  TO authenticated
  USING (is_admin() OR is_teacher());

-- Orang tua bisa lihat anak sendiri
CREATE POLICY "children_select_parent"
  ON children FOR SELECT
  TO authenticated
  USING (is_parent_of(id));

-- Admin bisa CRUD anak
CREATE POLICY "children_insert_admin"
  ON children FOR INSERT
  TO authenticated
  WITH CHECK (is_admin());

CREATE POLICY "children_update_admin"
  ON children FOR UPDATE
  TO authenticated
  USING (is_admin());

CREATE POLICY "children_delete_admin"
  ON children FOR DELETE
  TO authenticated
  USING (is_admin());

-- ============================================
-- PARENT_CHILDREN
-- ============================================
ALTER TABLE parent_children ENABLE ROW LEVEL SECURITY;

CREATE POLICY "parent_children_select"
  ON parent_children FOR SELECT
  TO authenticated
  USING (parent_id = auth.uid() OR is_admin());

CREATE POLICY "parent_children_insert"
  ON parent_children FOR INSERT
  TO authenticated
  WITH CHECK (is_admin());

CREATE POLICY "parent_children_delete"
  ON parent_children FOR DELETE
  TO authenticated
  USING (is_admin());

-- ============================================
-- PROGRAMS
-- ============================================
ALTER TABLE programs ENABLE ROW LEVEL SECURITY;

-- Publik bisa lihat program yang public
CREATE POLICY "programs_select_public"
  ON programs FOR SELECT
  TO anon
  USING (is_public = TRUE AND is_active = TRUE);

-- Authenticated bisa lihat semua program
CREATE POLICY "programs_select_authenticated"
  ON programs FOR SELECT
  TO authenticated
  USING (TRUE);

-- Admin bisa CRUD
CREATE POLICY "programs_modify_admin"
  ON programs FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- SESSIONS
-- ============================================
ALTER TABLE sessions ENABLE ROW LEVEL SECURITY;

-- Admin & pengajar bisa lihat semua sesi
CREATE POLICY "sessions_select_staff"
  ON sessions FOR SELECT
  TO authenticated
  USING (is_admin() OR is_teacher());

-- Pengajar bisa create & update sesi sendiri
CREATE POLICY "sessions_insert_teacher"
  ON sessions FOR INSERT
  TO authenticated
  WITH CHECK (is_admin() OR (is_teacher() AND teacher_id = auth.uid()));

CREATE POLICY "sessions_update_teacher"
  ON sessions FOR UPDATE
  TO authenticated
  USING (is_admin() OR (is_teacher() AND teacher_id = auth.uid()));

-- ============================================
-- ATTENDANCE
-- ============================================
ALTER TABLE attendance ENABLE ROW LEVEL SECURITY;

-- Staff bisa lihat semua attendance
CREATE POLICY "attendance_select_staff"
  ON attendance FOR SELECT
  TO authenticated
  USING (is_admin() OR is_teacher());

-- Orang tua bisa lihat attendance anak sendiri
CREATE POLICY "attendance_select_parent"
  ON attendance FOR SELECT
  TO authenticated
  USING (is_parent_of(child_id));

-- Staff bisa input attendance
CREATE POLICY "attendance_insert_staff"
  ON attendance FOR INSERT
  TO authenticated
  WITH CHECK (is_admin() OR is_teacher());

CREATE POLICY "attendance_update_staff"
  ON attendance FOR UPDATE
  TO authenticated
  USING (is_admin() OR is_teacher());

-- ============================================
-- LEAVE_REQUESTS
-- ============================================
ALTER TABLE leave_requests ENABLE ROW LEVEL SECURITY;

-- Orang tua bisa lihat & create izin untuk anak sendiri
CREATE POLICY "leave_select_parent"
  ON leave_requests FOR SELECT
  TO authenticated
  USING (parent_id = auth.uid() OR is_admin() OR is_teacher());

CREATE POLICY "leave_insert_parent"
  ON leave_requests FOR INSERT
  TO authenticated
  WITH CHECK (parent_id = auth.uid() AND is_parent_of(child_id));

-- Admin/pengajar bisa approve/reject
CREATE POLICY "leave_update_staff"
  ON leave_requests FOR UPDATE
  TO authenticated
  USING (is_admin() OR is_teacher());

-- ============================================
-- QURAN_PROGRESS
-- ============================================
ALTER TABLE quran_progress ENABLE ROW LEVEL SECURITY;

CREATE POLICY "quran_select_staff"
  ON quran_progress FOR SELECT
  TO authenticated
  USING (is_admin() OR is_teacher());

CREATE POLICY "quran_select_parent"
  ON quran_progress FOR SELECT
  TO authenticated
  USING (is_parent_of(child_id));

CREATE POLICY "quran_insert_staff"
  ON quran_progress FOR INSERT
  TO authenticated
  WITH CHECK (is_admin() OR is_teacher());

CREATE POLICY "quran_update_staff"
  ON quran_progress FOR UPDATE
  TO authenticated
  USING (is_admin() OR (is_teacher() AND teacher_id = auth.uid()));

-- ============================================
-- STUDY_PROGRESS
-- ============================================
ALTER TABLE study_progress ENABLE ROW LEVEL SECURITY;

CREATE POLICY "study_select_staff"
  ON study_progress FOR SELECT
  TO authenticated
  USING (is_admin() OR is_teacher());

CREATE POLICY "study_select_parent"
  ON study_progress FOR SELECT
  TO authenticated
  USING (is_parent_of(child_id));

CREATE POLICY "study_insert_staff"
  ON study_progress FOR INSERT
  TO authenticated
  WITH CHECK (is_admin() OR is_teacher());

CREATE POLICY "study_update_staff"
  ON study_progress FOR UPDATE
  TO authenticated
  USING (is_admin() OR (is_teacher() AND teacher_id = auth.uid()));

-- ============================================
-- PROGRESS_REPORTS
-- ============================================
ALTER TABLE progress_reports ENABLE ROW LEVEL SECURITY;

CREATE POLICY "reports_select_staff"
  ON progress_reports FOR SELECT
  TO authenticated
  USING (is_admin() OR is_teacher());

CREATE POLICY "reports_select_parent"
  ON progress_reports FOR SELECT
  TO authenticated
  USING (is_parent_of(child_id) AND is_published = TRUE);

CREATE POLICY "reports_modify_admin"
  ON progress_reports FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- FEE_TYPES
-- ============================================
ALTER TABLE fee_types ENABLE ROW LEVEL SECURITY;

CREATE POLICY "fee_types_select"
  ON fee_types FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "fee_types_modify_admin"
  ON fee_types FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- FEE_INVOICES
-- ============================================
ALTER TABLE fee_invoices ENABLE ROW LEVEL SECURITY;

CREATE POLICY "invoices_select_admin"
  ON fee_invoices FOR SELECT
  TO authenticated
  USING (is_admin());

CREATE POLICY "invoices_select_parent"
  ON fee_invoices FOR SELECT
  TO authenticated
  USING (is_parent_of(child_id));

CREATE POLICY "invoices_modify_admin"
  ON fee_invoices FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- DONATIONS
-- ============================================
ALTER TABLE donations ENABLE ROW LEVEL SECURITY;

-- Publik bisa lihat donasi terverifikasi (untuk transparansi)
CREATE POLICY "donations_select_public"
  ON donations FOR SELECT
  TO anon
  USING (is_verified = TRUE);

CREATE POLICY "donations_select_authenticated"
  ON donations FOR SELECT
  TO authenticated
  USING (is_verified = TRUE OR donor_id = auth.uid() OR is_admin());

-- Authenticated bisa create donasi
CREATE POLICY "donations_insert_authenticated"
  ON donations FOR INSERT
  TO authenticated
  WITH CHECK (TRUE);

-- Anon juga bisa create donasi (donasi publik tanpa login)
CREATE POLICY "donations_insert_anon"
  ON donations FOR INSERT
  TO anon
  WITH CHECK (TRUE);

-- Admin bisa verify
CREATE POLICY "donations_update_admin"
  ON donations FOR UPDATE
  TO authenticated
  USING (is_admin());

-- ============================================
-- EXPENSES
-- ============================================
ALTER TABLE expenses ENABLE ROW LEVEL SECURITY;

-- Publik bisa lihat pengeluaran (transparansi)
CREATE POLICY "expenses_select_public"
  ON expenses FOR SELECT
  TO anon
  USING (TRUE);

CREATE POLICY "expenses_select_authenticated"
  ON expenses FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "expenses_modify_admin"
  ON expenses FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- PROJECTS
-- ============================================
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

CREATE POLICY "projects_select_public"
  ON projects FOR SELECT
  TO anon
  USING (is_public = TRUE);

CREATE POLICY "projects_select_authenticated"
  ON projects FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "projects_modify_admin"
  ON projects FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- PROJECT_MILESTONES
-- ============================================
ALTER TABLE project_milestones ENABLE ROW LEVEL SECURITY;

CREATE POLICY "milestones_select_public"
  ON project_milestones FOR SELECT
  TO anon
  USING (EXISTS (SELECT 1 FROM projects WHERE projects.id = project_id AND is_public = TRUE));

CREATE POLICY "milestones_select_authenticated"
  ON project_milestones FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "milestones_modify_admin"
  ON project_milestones FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- PROJECT_UPDATES
-- ============================================
ALTER TABLE project_updates ENABLE ROW LEVEL SECURITY;

CREATE POLICY "updates_select_public"
  ON project_updates FOR SELECT
  TO anon
  USING (EXISTS (SELECT 1 FROM projects WHERE projects.id = project_id AND is_public = TRUE));

CREATE POLICY "updates_select_authenticated"
  ON project_updates FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "updates_modify_admin"
  ON project_updates FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- PROJECT_UPDATE_PHOTOS
-- ============================================
ALTER TABLE project_update_photos ENABLE ROW LEVEL SECURITY;

CREATE POLICY "photos_select_public"
  ON project_update_photos FOR SELECT
  TO anon
  USING (EXISTS (
    SELECT 1 FROM project_updates pu
    JOIN projects p ON p.id = pu.project_id
    WHERE pu.id = update_id AND p.is_public = TRUE
  ));

CREATE POLICY "photos_select_authenticated"
  ON project_update_photos FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "photos_modify_admin"
  ON project_update_photos FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- ANNOUNCEMENTS
-- ============================================
ALTER TABLE announcements ENABLE ROW LEVEL SECURITY;

CREATE POLICY "announcements_select_public"
  ON announcements FOR SELECT
  TO anon
  USING (is_public = TRUE AND published_at IS NOT NULL AND published_at <= NOW());

CREATE POLICY "announcements_select_authenticated"
  ON announcements FOR SELECT
  TO authenticated
  USING (
    (published_at IS NOT NULL AND published_at <= NOW()
    AND (target_roles IS NULL OR get_user_roles() && target_roles))
    OR is_admin()
  );

CREATE POLICY "announcements_modify_admin"
  ON announcements FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- GALLERY
-- ============================================
ALTER TABLE gallery ENABLE ROW LEVEL SECURITY;

CREATE POLICY "gallery_select_public"
  ON gallery FOR SELECT
  TO anon
  USING (is_public = TRUE);

CREATE POLICY "gallery_select_authenticated"
  ON gallery FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "gallery_modify_admin"
  ON gallery FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- NOTIFICATIONS
-- ============================================
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

CREATE POLICY "notifications_select_own"
  ON notifications FOR SELECT
  TO authenticated
  USING (user_id = auth.uid());

CREATE POLICY "notifications_update_own"
  ON notifications FOR UPDATE
  TO authenticated
  USING (user_id = auth.uid());

-- Service role inserts notifications (via Edge Functions)
CREATE POLICY "notifications_insert_service"
  ON notifications FOR INSERT
  TO service_role
  WITH CHECK (TRUE);

-- Admin juga bisa insert notifications
CREATE POLICY "notifications_insert_admin"
  ON notifications FOR INSERT
  TO authenticated
  WITH CHECK (is_admin());

-- ============================================
-- ARTICLES
-- ============================================
ALTER TABLE articles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "articles_select_public"
  ON articles FOR SELECT
  TO anon
  USING (status = 'published' AND published_at IS NOT NULL AND published_at <= NOW());

CREATE POLICY "articles_select_authenticated"
  ON articles FOR SELECT
  TO authenticated
  USING (status = 'published' OR is_admin());

CREATE POLICY "articles_modify_admin"
  ON articles FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- PAGES
-- ============================================
ALTER TABLE pages ENABLE ROW LEVEL SECURITY;

CREATE POLICY "pages_select_public"
  ON pages FOR SELECT
  TO anon
  USING (TRUE);

CREATE POLICY "pages_select_authenticated"
  ON pages FOR SELECT
  TO authenticated
  USING (TRUE);

CREATE POLICY "pages_modify_admin"
  ON pages FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- INVITATIONS
-- ============================================
ALTER TABLE invitations ENABLE ROW LEVEL SECURITY;

CREATE POLICY "invitations_select_admin"
  ON invitations FOR SELECT
  TO authenticated
  USING (is_admin());

CREATE POLICY "invitations_modify_admin"
  ON invitations FOR ALL
  TO authenticated
  USING (is_admin())
  WITH CHECK (is_admin());

-- ============================================
-- SUBJECTS, CLASSES, CLASS_ENROLLMENTS, SCHEDULES
-- ============================================
ALTER TABLE subjects ENABLE ROW LEVEL SECURITY;
ALTER TABLE classes ENABLE ROW LEVEL SECURITY;
ALTER TABLE class_enrollments ENABLE ROW LEVEL SECURITY;
ALTER TABLE schedules ENABLE ROW LEVEL SECURITY;

CREATE POLICY "subjects_select" ON subjects FOR SELECT TO authenticated USING (TRUE);
CREATE POLICY "subjects_modify_admin" ON subjects FOR ALL TO authenticated USING (is_admin()) WITH CHECK (is_admin());

CREATE POLICY "classes_select" ON classes FOR SELECT TO authenticated USING (TRUE);
CREATE POLICY "classes_modify_admin" ON classes FOR ALL TO authenticated USING (is_admin()) WITH CHECK (is_admin());

CREATE POLICY "enrollments_select" ON class_enrollments FOR SELECT TO authenticated USING (TRUE);
CREATE POLICY "enrollments_modify_admin" ON class_enrollments FOR ALL TO authenticated USING (is_admin()) WITH CHECK (is_admin());

CREATE POLICY "schedules_select_public" ON schedules FOR SELECT TO anon USING (TRUE);
CREATE POLICY "schedules_select" ON schedules FOR SELECT TO authenticated USING (TRUE);
CREATE POLICY "schedules_modify_admin" ON schedules FOR ALL TO authenticated USING (is_admin()) WITH CHECK (is_admin());
```

---

## Database Functions & Triggers

### Auto-update `updated_at`

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables with updated_at
CREATE TRIGGER set_updated_at BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON children
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON programs
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON classes
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON sessions
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON attendance
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON quran_progress
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON study_progress
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON progress_reports
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON fee_types
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON fee_invoices
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON projects
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON project_milestones
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON project_updates
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON announcements
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON articles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER set_updated_at BEFORE UPDATE ON pages
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### Auto-create profile on auth signup

```sql
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO profiles (id, full_name, phone, roles)
  VALUES (
    NEW.id,
    COALESCE(NEW.raw_user_meta_data->>'full_name', 'User'),
    NEW.phone,
    COALESCE(
      (NEW.raw_user_meta_data->>'roles')::user_role[],
      ARRAY['orangtua']::user_role[]
    )
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW
  EXECUTE FUNCTION handle_new_user();
```

### Update project collected_amount on donation verify

```sql
CREATE OR REPLACE FUNCTION update_project_collected()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.project_id IS NOT NULL AND NEW.is_verified = TRUE THEN
    UPDATE projects
    SET collected_amount = (
      SELECT COALESCE(SUM(amount), 0)
      FROM donations
      WHERE project_id = NEW.project_id AND is_verified = TRUE
    )
    WHERE id = NEW.project_id;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_donation_verified
  AFTER INSERT OR UPDATE ON donations
  FOR EACH ROW
  EXECUTE FUNCTION update_project_collected();
```

---

## Entity Relationship Diagram (Text)

```
profiles ─────┐
  │            │
  │ 1:N        │ N:1
  │            │
parent_children ─── children
                      │
                      │ 1:N
                      ├── class_enrollments ── classes ── programs
                      │                         │
                      │                         │ 1:N
                      │                         ├── schedules
                      │                         └── sessions ── attendance
                      │                                │
                      ├── leave_requests               │
                      ├── quran_progress ──────────────┘
                      ├── study_progress ── subjects
                      ├── progress_reports
                      └── fee_invoices ── fee_types

profiles ─── donations ── projects
                           │
                           ├── project_milestones
                           ├── project_updates ── project_update_photos
                           └── expenses

profiles ─── announcements
profiles ─── gallery
profiles ─── notifications
profiles ─── articles
profiles ─── pages
profiles ─── invitations
```
