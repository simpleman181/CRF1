# CRF App — Setup Guide
## Go live in ~20 minutes, completely free

---

## What You're Deploying

| File | Purpose |
|---|---|
| `index.html` | CRF data entry form (used by all centres) |
| `admin.html` | Admin dashboard (Mumbai team only) |
| `config.js` | Your Supabase credentials (fill this in) |

---

## STEP 1 — Create Your Supabase Database (5 min)

1. Go to **https://supabase.com** → click **Start your project** (free)
2. Sign up with Google or GitHub
3. Click **New Project** → give it a name (e.g. `crf-study`) → set a database password → choose region **South Asia (Mumbai)** → click **Create Project**
4. Wait ~2 minutes for it to spin up

### Create the database table

5. In your Supabase project, click **SQL Editor** in the left sidebar
6. Paste this SQL and click **Run**:

```sql
CREATE TABLE crf_submissions (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  submitted_at timestamptz DEFAULT now(),
  study_name text, study_id text, centre_id text, location text,
  investigator_name text, clinic_name text,
  mobile text, email text, crc_name text,
  subject_id text, subject_initials text, sex text,
  dob text, enrol_date text, age_months text, age_years text,
  weight_kg text, height_cm text,
  pe_general text, pe_cvs text, pe_rs text, pe_git text, pe_cns text,
  v1_weight text, v1_bmi text, v1_pulse text, v1_temp text, v1_rr text,
  d1_fever text, d1_chills text, d1_headache text, d1_cough text,
  d1_rhinitis text, d1_vomit text, d1_inapp text,
  d1_sorethroat text, d1_phareryt text, d1_tonsil text,
  d1_exudate text, d1_lymph text,
  d1_bulge text, d1_tmmob text, d1_otalgia text,
  d1_airfluid text, d1_erythtm text,
  total_baseline text,
  v7_weight text, v7_bmi text, v7_pulse text, v7_temp text, v7_rr text,
  d7_fever text, d7_chills text, d7_headache text, d7_cough text,
  d7_rhinitis text, d7_vomit text, d7_inapp text,
  d7_sorethroat text, d7_phareryt text, d7_tonsil text,
  d7_exudate text, d7_lymph text,
  d7_bulge text, d7_tmmob text, d7_otalgia text,
  d7_airfluid text, d7_erythtm text,
  total_eot text, pct_reduction text,
  ae_occurred text, adverse_events text,
  comp_dose_1 text, comp_dose_2 text, comp_dose_3 text,
  comp_dose_4 text, comp_dose_5 text,
  comp_correct text, comp_adherent text,
  palatability text,
  concom_paracetamol int, concom_cpm int, concom_broncho int,
  concom_nasal int, concom_ear int,
  concomitant_other text,
  status text DEFAULT 'submitted'
);

-- Allow anyone to INSERT (centres submit data)
-- Only authenticated users can SELECT (admin reads data)
ALTER TABLE crf_submissions ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Allow insert for all" ON crf_submissions
  FOR INSERT WITH CHECK (true);

CREATE POLICY "Allow select for anon" ON crf_submissions
  FOR SELECT USING (true);

CREATE POLICY "Allow delete for anon" ON crf_submissions
  FOR DELETE USING (true);
```

### Get your API keys

7. Go to **Settings → API** in the left sidebar
8. Copy:
   - **Project URL** → looks like `https://abcdefgh.supabase.co`
   - **anon / public** key → long string starting with `eyJ...`

---

## STEP 2 — Update config.js (1 min)

Open `config.js` and replace the placeholder values:

```javascript
const SUPABASE_URL  = "https://YOUR_PROJECT_ID.supabase.co";   // ← paste Project URL
const SUPABASE_ANON = "eyJhbGci...";                            // ← paste anon key

// Change this to your preferred admin password
const ADMIN_PASSWORD = "CRFAdmin@2024";
```

---

## STEP 3 — Deploy to Vercel (5 min)

### Option A — Drag & Drop (easiest, no GitHub needed)

1. Go to **https://vercel.com** → sign up free
2. Click **Add New → Project**
3. Click **"Deploy from your computer"** or drag the folder containing all 3 files
4. Click **Deploy**
5. In ~30 seconds you get a live URL like `https://crf-study.vercel.app`

### Option B — Via GitHub

1. Create a GitHub repo → upload all 3 files
2. On Vercel → **Import Git Repository** → select your repo
3. Click **Deploy**

### Your live URLs will be:
- **CRF Form** (share with all centres): `https://your-app.vercel.app/index.html`
- **Admin Dashboard** (Mumbai only): `https://your-app.vercel.app/admin.html`

---

## STEP 4 — Share with Centres

Send each centre coordinator this one link:
```
https://your-app.vercel.app/index.html
```

They open it in any browser — no installation needed.
Each form submission instantly appears in your admin dashboard.

---

## Admin Dashboard

Login at `https://your-app.vercel.app/admin.html`

**Password**: whatever you set in `config.js` (default: `CRFAdmin@2024`)

**Features:**
- Live count of total submissions, active centres, average baseline score, AE count
- Per-centre breakdown with enrollment count and last submission time
- Full data table with search, filter by centre, filter by AE status
- Sort by any column
- Click **View** on any row to see all details for that subject
- **Export All** downloads every submission as a CSV for Excel/SPSS

---

## Costs

| Service | Plan | Cost |
|---|---|---|
| Supabase | Free tier | **₹0** (500MB database, 50,000 rows) |
| Vercel | Hobby plan | **₹0** (unlimited deployments) |

Both free tiers are more than sufficient for a multi-centre clinical study.

---

## Updating the CRF Form Later

If you need to add fields:
1. Add the field to `index.html`
2. Add the column to Supabase via SQL: `ALTER TABLE crf_submissions ADD COLUMN new_field text;`
3. Add it to the `collectFormData()` function in `index.html`
4. Re-deploy to Vercel (if using GitHub, just push the change)

---

## Troubleshooting

| Problem | Fix |
|---|---|
| "Connection error" in admin | Check SUPABASE_URL and SUPABASE_ANON in config.js |
| Submit fails with 401 | Your anon key is wrong — re-copy from Supabase Settings → API |
| Submit fails with 404 | Table doesn't exist — re-run the SQL in Step 1 |
| Can't see data in admin | Check Row Level Security policies were created correctly |
