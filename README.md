# Investment Behaviour Study — Survey App

A premium, mobile-first survey for the research project *"Mathematical Modelling of
Student's Investment Behaviour."* Built with Next.js (App Router), Tailwind CSS, and
Supabase. Includes conditional branching (Section C vs D based on Q7), validation,
autosave drafts, dark/light mode, and a passcode-protected admin dashboard with charts
and CSV export.

## Stack

- **Next.js 14** (App Router) + React 18
- **Tailwind CSS** for styling
- **Supabase** (Postgres) for storing responses
- **Recharts** for the admin dashboard
- **Lucide** icons

## 1. Local setup

```bash
npm install
cp .env.example .env.local
```

Fill in `.env.local` with your Supabase values (see step 2) and pick an admin passcode.

```bash
npm run dev
```

Visit `http://localhost:3000`.

## 2. Set up Supabase

1. Create a free project at [supabase.com](https://supabase.com).
2. Go to **SQL Editor → New query**, paste the contents of `supabase/schema.sql`,
   and run it. This creates the `responses` table and locks it down with Row Level
   Security so the browser can only insert new responses, never read or edit them.
3. Go to **Project Settings → API** and copy:
   - `Project URL` → `NEXT_PUBLIC_SUPABASE_URL`
   - `anon public` key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `service_role` key → `SUPABASE_SERVICE_ROLE_KEY` (keep this secret — it's only
     used inside server-side API routes, never sent to the browser)

## 3. How the pieces fit together

```
app/
  page.js                 landing page
  survey/page.js           the question flow (branches after Q7)
  thank-you/page.js        confirmation screen
  admin/page.js             passcode entry
  admin/dashboard/page.js   charts, table, search, CSV export
  api/responses/route.js    POST = public submit, GET = admin-only read
  api/admin-auth/route.js   checks ADMIN_PASSCODE, sets an httpOnly cookie
  api/export/route.js       admin-only CSV download
lib/
  questions.js              the single source of truth for all questions
  supabaseClient.js         public client (browser-safe, anon key)
  supabaseAdmin.js          server-only client (service role key)
supabase/schema.sql          run this once in Supabase's SQL editor
```

**To add, edit, remove, or reorder questions:** everything lives in `lib/questions.js`
as plain arrays (`SECTION_B`, `SECTION_C`, `SECTION_D`, `SECTION_E`). Add a new column
in Supabase (`alter table responses add column q16 text;`) and add it to `ALL_COLUMNS`
in the same file, and it'll flow through the form, the API, and the dashboard
automatically. There's no drag-and-drop question builder in this version — this is the
fastest path for a project on a deadline. Ask if you'd like that built as a next step.

## 4. Deploy to Vercel

1. Push this folder to a GitHub repository.
2. Go to [vercel.com/new](https://vercel.com/new) and import the repo.
3. In **Environment Variables**, add the same four values from your `.env.local`
   (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`,
   `SUPABASE_SERVICE_ROLE_KEY`, `ADMIN_PASSCODE`).
4. Click **Deploy**. Vercel gives you a public URL — anyone can open it in Chrome,
   Safari, Edge, or Firefox with no login and no app install.

## 5. Using the admin dashboard

Visit `/admin` on your deployed site, enter your `ADMIN_PASSCODE`, and you'll land on
`/admin/dashboard`: total responses, investor vs non-investor split, allowance and
investment-channel breakdowns, reasons for not investing, and math-awareness charts,
plus a searchable response table and a **CSV export** button (opens `/api/export`).

The passcode is checked server-side and stored as an httpOnly session cookie — it's
never exposed in the browser's JavaScript. For a bigger deployment (multiple admins,
audit logs), swap this for Supabase Auth.

## 6. Research statistics (mean, median, mode, cross-tabulation)

The dashboard covers the descriptive charts asked for. For deeper statistics — mean,
median, mode, cross-tabulation between variables — the cleanest path is exporting the
CSV and running it through Excel/Google Sheets pivot tables, or a Python notebook
(`pandas.describe()`, `pandas.crosstab()`). Ask if you'd like a ready-made analysis
notebook or an Excel template wired to the CSV columns.

## Known limitations (by design, to keep this shippable)

- No question-builder UI — edit `lib/questions.js` directly.
- Admin auth is a single shared passcode, not per-user accounts.
- No automated PDF report yet (CSV export is wired up; PDF can be added).
