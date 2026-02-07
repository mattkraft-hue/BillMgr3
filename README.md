# BCBA Billing Tracker

A mobile-optimized web app for BCBA behavior analysts to track billing time with clients using Indiana's waiver system. **Now with persistent cloud storage via Supabase.**

## Features

- **Persistent Cloud Storage** — All data stored in Supabase (PostgreSQL). Syncs across devices. Never lose data.
- **Secure Auth** — Real email/password authentication via Supabase Auth (no email confirmation required).
- **Row Level Security** — Each user can only see their own data, enforced at the database level.
- **Client Management** — Medicaid ID, diagnosis, waiver type, authorization numbers, monthly unit budgets.
- **Billing Records** — Indiana waiver service codes (97151–97158, H0031, H2014, H2019, T1024), auto-calculated units.
- **Budget Tracking** — Visual progress bars for monthly unit usage per client.
- **File Attachments** — Upload PDFs, images, documents attached to each billing record.
- **Excel Import/Export** — Monthly billing data and budget summaries.
- **iOS-Style UI** — Native iOS design with blur effects and smooth animations.

## Quick Setup (5 Minutes)

### 1. Create Supabase Project
1. Go to [supabase.com](https://supabase.com) and create a free account
2. Create a new project (choose any name, set a DB password, pick a region)
3. Wait for the project to finish provisioning (~1 minute)

### 2. Create Database Tables
1. In your Supabase dashboard, go to **SQL Editor**
2. Paste and run this SQL:

```sql
CREATE TABLE IF NOT EXISTS clients (
  id TEXT PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  client_ext_id TEXT DEFAULT '',
  dob DATE,
  diagnosis TEXT DEFAULT '',
  monthly_budget_units INTEGER DEFAULT 0,
  waiver_type TEXT DEFAULT 'FSW',
  auth_number TEXT DEFAULT '',
  notes TEXT DEFAULT '',
  created_at TIMESTAMPTZ DEFAULT now()
);
ALTER TABLE clients ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users own clients" ON clients FOR ALL USING (auth.uid() = user_id);

CREATE TABLE IF NOT EXISTS records (
  id TEXT PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  client_id TEXT NOT NULL REFERENCES clients(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  service_code TEXT NOT NULL,
  service_name TEXT DEFAULT '',
  start_time TEXT DEFAULT '',
  end_time TEXT DEFAULT '',
  minutes INTEGER DEFAULT 0,
  units INTEGER DEFAULT 0,
  location TEXT DEFAULT '',
  status TEXT DEFAULT 'pending',
  notes TEXT DEFAULT '',
  created_at TIMESTAMPTZ DEFAULT now()
);
ALTER TABLE records ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users own records" ON records FOR ALL USING (auth.uid() = user_id);

CREATE TABLE IF NOT EXISTS files (
  id TEXT PRIMARY KEY,
  record_id TEXT NOT NULL REFERENCES records(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  type TEXT DEFAULT '',
  size INTEGER DEFAULT 0,
  data TEXT,
  uploaded_at TIMESTAMPTZ DEFAULT now()
);
ALTER TABLE files ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users own files" ON files FOR ALL
  USING (EXISTS (SELECT 1 FROM records WHERE records.id = files.record_id AND records.user_id = auth.uid()));
```

### 3. Disable Email Confirmation
1. In your Supabase dashboard, go to **Authentication → Settings → Email**
2. **Turn OFF** "Confirm email"
3. This allows users to sign in immediately after creating an account

### 4. Get API Credentials
1. Go to **Settings → API** in your Supabase dashboard
2. Copy your **Project URL** and **anon public key**

### 5. Deploy to GitHub Pages
1. Create a new GitHub repository
2. Upload `index.html`, `manifest.json`, and `README.md`
3. Go to **Settings → Pages** → select `main` branch → Save
4. Your app will be live at `https://yourusername.github.io/repo-name/`

### 6. Connect the App
1. Open your deployed app
2. Paste your Supabase URL and anon key on the setup screen
3. Click **Connect & Continue**
4. Create your account and start billing!

## How Storage Works

| Feature | Details |
|---------|---------|
| **Database** | Supabase (PostgreSQL) — free tier: 500MB, unlimited API requests |
| **Auth** | Supabase Auth — email/password, no email confirmation required |
| **Security** | Row Level Security — users only see their own data |
| **Sync** | Real-time sync across all devices and browsers |
| **Files** | Stored as base64 in the database (suitable for session notes/small docs) |

## License

MIT
