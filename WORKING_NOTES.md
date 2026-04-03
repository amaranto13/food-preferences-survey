# Working Notes — Food Preferences Survey

> **Internal document. Not public-facing. For developer and AI assistant use only.**
> Update this file at the end of every working session before closing.

---

## How to Use This File (For AI Assistants)

1. Read this entire file before taking any action on this project.
2. Read `README.md` for public-facing context, installation steps, and the Supabase SQL schema.
3. Do not change the folder structure or any naming conventions without first discussing the change with the developer.
4. Follow all conventions in the **Conventions** section exactly — do not introduce new patterns without explicit approval.
5. Do not suggest anything listed in **What Was Tried and Rejected**. Those decisions are final.
6. Ask before making any large structural changes (routing library swap, state management addition, CSS framework change, etc.).
7. This project was built with significant AI assistance. Refactor conservatively — prefer targeted edits over wholesale rewrites.

---

## Current State

**Last Updated:** 2026-03-26

The app is complete and functional. All three pages (Home, Survey, Results) are built, routed, and styled. The Supabase integration works end-to-end once the user runs the SQL schema in the Supabase dashboard. Azure Static Web Apps routing config is in place. A full public README and this internal working-notes document are at the project root.

### What Is Working

- [x] Home page with "Take the Survey" and "View Results" navigation
- [x] Four-question survey form with client-side validation
- [x] Supabase insert on form submission
- [x] Thank-you screen with disabled "Response Submitted" button and "View Results" link
- [x] Results page with total count and three Recharts bar charts
- [x] "Other" food normalization (lowercase + trim for counting, title case for display)
- [x] Shared Footer component on all three pages
- [x] ARIA labels, `aria-describedby`, and `aria-invalid` on all form controls
- [x] `staticwebapp.config.json` for Azure Static Web Apps SPA routing fallback
- [x] Environment variables wired via `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`
- [x] `README.md` with badges, SQL schema, Azure deployment guide, and project structure
- [x] `WORKING_NOTES.md` (this file)

### What Is Partially Built

- [ ] Results aggregation — works correctly but loads all rows on every page visit (no pagination or caching)
- [ ] "Other" food normalization — handles case and whitespace but not typos (e.g. "Tacos" vs "Taco" count separately)

### What Is Not Started

- [ ] Duplicate-submission prevention
- [ ] Admin/export view for raw responses
- [ ] Any form of authentication or session tracking
- [ ] Automated tests (unit or e2e)

---

## Current Task

The app is in a complete and deployable state as of the last session. The final steps taken were adding `staticwebapp.config.json` for Azure deployment and writing `README.md` and `WORKING_NOTES.md`.

**Next step:** The developer needs to run the Supabase SQL schema (in `README.md`) against their Supabase project to activate form submission and results. No code changes are required.

---

## Architecture and Tech Stack

| Technology | Version | Why It Was Chosen |
|---|---|---|
| React | 19.1.0 | UI component framework; required by the Replit react-vite scaffold |
| TypeScript | ^5.x | Static typing; required by the scaffold template |
| Vite | ^7.3.0 | Dev server and bundler; included in scaffold, fast HMR |
| Tailwind CSS | ^4.x | Utility-first styling; included in scaffold |
| shadcn/ui + Radix UI | various ^1–2.x | Accessible primitives; included in scaffold — not individually chosen |
| wouter | ^3.3.5 | Lightweight client-side routing; included in scaffold template default |
| Recharts | ^2.15.2 | Bar charts on Results page; pre-installed in scaffold |
| @supabase/supabase-js | ^2.100.0 | Supabase client; added manually for DB integration |
| pnpm workspaces | 8+ | Monorepo package management; imposed by Replit project structure |

---

## Project Structure Notes

```
artifacts/food-survey/
├── public/                        # Static assets (currently empty)
├── src/
│   ├── pages/
│   │   ├── Home.tsx               # Landing page; CTA buttons to /survey and /results
│   │   ├── Survey.tsx             # Form + validation + Supabase insert + thank-you state
│   │   └── Results.tsx            # Fetches all rows, aggregates in-browser, renders charts
│   ├── components/
│   │   └── Footer.tsx             # Rendered at bottom of every page
│   ├── lib/
│   │   └── supabase.ts            # Supabase client — reads VITE_ env vars at runtime
│   ├── App.tsx                    # wouter <Switch> with /, /survey, /results routes
│   └── index.css                  # Tailwind base + CSS custom properties for theme
├── staticwebapp.config.json       # Azure SWA SPA fallback — do not move or rename
├── vite.config.ts                 # Dev server config (host: 0.0.0.0 for Replit proxy)
└── package.json                   # Dependencies and npm scripts
```

### Non-Obvious Decisions

- `staticwebapp.config.json` lives at the **app root** (`artifacts/food-survey/`), not in `public/`. Azure SWA reads it from `app_location`. It does not need to be in `dist/`.
- All data aggregation (grouping by food, meal, frequency) happens **in the browser** in `Results.tsx`. There are no database views or server-side queries.
- The primary brand color `#8A3BDB` is written as an **inline Tailwind arbitrary value** (`bg-[#8A3BDB]`), not as a CSS variable or Tailwind config extension.
- `Survey.tsx` manages submission state with a local `submitted` boolean. On `true`, the form is replaced by a thank-you block — no navigation occurs.

### Files That Must Not Be Changed Without Discussion

- `src/lib/supabase.ts` — changing the client initialization will break all DB calls
- `staticwebapp.config.json` — required for Azure SWA SPA routing; removing it causes 404s on deep links
- `src/index.css` — contains the Tailwind base import and CSS custom properties that control the entire visual theme
- `vite.config.ts` — `host: true` is required for the Replit iframe proxy to work

---

## Data / Database

**Database:** Supabase (external hosted PostgreSQL)

### Table: `survey_responses`

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | uuid | auto | `gen_random_uuid()` default; primary key |
| `created_at` | timestamptz | auto | `now()` default; set by DB |
| `favorite_food` | text | yes | Free-text Q1 answer |
| `common_meal` | text | yes | One of: Breakfast, Lunch, Dinner, Snacks |
| `takeout_frequency` | text | yes | One of: Never, 1–2 times per week, 3–4 times per week, 5+ times per week |
| `enjoyed_foods` | text[] | yes | Array of checked foods; always contains at least one entry |
| `other_food` | text | no | Populated only when "Other" is in `enjoyed_foods`; null otherwise |

**Row Level Security:** enabled. Two policies exist — public INSERT (no auth) and public SELECT (no auth).

**No migrations system.** Schema is applied manually via Supabase SQL Editor. The full SQL is in `README.md` under "Supabase Setup".

---

## Conventions

### Naming Conventions

- **Files:** PascalCase for React components (`Survey.tsx`, `Footer.tsx`), camelCase for utilities (`supabase.ts`)
- **Variables/functions:** camelCase throughout
- **CSS classes:** Tailwind utility strings only; no custom class names except those generated by shadcn/ui
- **Supabase table columns:** `snake_case` to match PostgreSQL conventions

### Code Style

- TypeScript strict mode; no `any` types
- Functional components only; no class components
- Props typed inline with `interface` or `type` as needed
- State managed with `useState`; no external state library
- No default exports from utility files (e.g. `supabase.ts` uses named export)

### Framework Patterns

- Routing: `useLocation` hook from wouter for programmatic navigation; `<Link>` for declarative links
- All three pages follow the same layout wrapper: `<div className="min-h-screen flex flex-col">` with `<main>` and `<Footer />` at bottom
- Error messages: rendered as `<p id="[field]-error" role="alert">` and referenced via `aria-describedby` on the input
- Form state: single `form` object via `useState`; spread updates (`setForm(prev => ({ ...prev, field: val }))`)

### Git Commit Style

- Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`
- Keep subject line under 72 characters
- Reference the question or feature area in the message (e.g. `fix: correct aria-invalid on radio group`)

---

## Decisions and Tradeoffs

- **wouter instead of react-router-dom:** wouter is the Replit react-vite scaffold default. It has an identical API for this use case. Do not suggest switching to react-router-dom.
- **VITE_ env vars as shared env vars (not secrets):** The Supabase anon key is a publishable key intentionally designed for browser use. Using `VITE_` prefix makes it available in the client bundle. This is correct and intentional — not a security issue.
- **Aggregation in the browser:** All `GROUP BY` logic runs client-side in `Results.tsx`. Avoids database views or Edge Functions for a simple class project. Acceptable at expected data volumes (<500 rows).
- **No deduplication:** Users can submit the survey multiple times. Acceptable scope for a class survey; not a production requirement.
- **shadcn/ui retained from scaffold:** The scaffold imports from `@/components/ui/*`. These files must remain even if their components aren't all used — removing them breaks imports.
- **Primary color as inline arbitrary value:** `#8A3BDB` is not added to `tailwind.config`. Arbitrary values keep the config clean for a project of this size.
- **`staticwebapp.config.json` at app root, not `public/`:** Azure SWA reads it from `app_location` before the build. Placing it in `public/` would also work but is redundant.

---

## What Was Tried and Rejected

- **`<span role="status">` for the "Response Submitted" element:** Initially implemented as a styled span. Rejected by code review — replaced with a proper `<button disabled aria-disabled="true">`. Do not revert to a span.
- **`staticwebapp.config.json` placed inside `public/`:** Discussed but not implemented. The file works correctly at the app root. Do not move it to `public/` — it adds no benefit and creates confusion about where Azure reads it from.
- **react-router-dom for routing:** The scaffold uses wouter. Installing react-router-dom would add a redundant dependency and require a full rewrite of `App.tsx` and all navigation hooks. Rejected. Do not suggest this.
- **Database views or Supabase RPC functions for aggregation:** Considered for the Results page. Rejected — unnecessary complexity for a class project with low expected data volume. Aggregation in the browser is sufficient.

---

## Known Issues and Workarounds

### Issue 1: Results page fails with "Could not load results" before SQL is run

- **Problem:** The `survey_responses` table does not exist until the developer manually runs the SQL in Supabase.
- **Workaround:** The Results page catches the Supabase error and displays a user-friendly message. No code fix needed — the user must run the SQL.
- **Do not remove** the try/catch error boundary in `Results.tsx`.

### Issue 2: Multiple submissions per user

- **Problem:** There is no mechanism preventing the same user from submitting the form multiple times.
- **Workaround:** None currently. A `localStorage` token approach is planned in the roadmap.
- **Do not add** any submission gating without discussing the approach first.

### Issue 3: "Other" entries with typos create separate chart bars

- **Problem:** `Results.tsx` normalizes "Other" entries via `toLowerCase().trim()` but cannot detect typos (e.g. "Tacos" vs "Taco").
- **Workaround:** None. Accepted limitation at current scope.
- **Do not remove** the normalization logic — it still helps with case and whitespace differences.

---

## Browser / Environment Compatibility

### Front End

- **Tested in:** Chromium-based browsers (via Replit preview iframe)
- **Expected support:** All modern browsers (Chrome, Firefox, Safari, Edge) — no IE support required
- **Known incompatibilities:** None identified
- **CSS:** Uses Tailwind CSS v4 with modern CSS features (`:where()`, logical properties); requires evergreen browsers

### Back End / Environment

- **Runtime:** Node.js 18+ (Replit-managed)
- **Package manager:** pnpm 8+ with workspace protocol
- **OS:** Linux (NixOS on Replit)
- **Deployment targets:** Replit (development and demo), Azure Static Web Apps (production)
- **Environment variables required at build time:** `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`

---

## Open Questions

- Should the Results page implement basic caching (e.g. `staleTime` via React Query) to avoid a full re-fetch on every visit?
- Should duplicate submission prevention use `localStorage` (simple) or a Supabase-level unique constraint on a session token column?
- Should the "Other" food input be validated against a minimum character count to reduce low-quality entries?
- Is a production deployment to Azure SWA planned, or will Replit remain the long-term host?

---

## Session Log

### 2026-03-26

**Accomplished:**
- Built the complete app from scratch: Home, Survey, and Results pages with full routing, Supabase integration, Recharts charts, and accessible form controls
- Added `staticwebapp.config.json` for Azure Static Web Apps SPA routing
- Addressed code review feedback: replaced `<span role="status">` with a proper disabled `<button>`; added per-control `aria-describedby` and `aria-invalid` to radio and checkbox inputs
- Wrote `README.md` with badges, full Supabase SQL, Azure deployment guide, known issues, roadmap, and acknowledgements
- Wrote `WORKING_NOTES.md` (this file)

**Left Incomplete:**
- Duplicate submission prevention (not in scope for v1.0)
- Results page pagination / caching

**Decisions Made:**
- Keep wouter (scaffold default); do not switch to react-router-dom
- VITE_ env vars as shared env vars, not secrets — intentionally public
- Aggregation in-browser; no database views

**Next Step:**
Developer must run the Supabase SQL schema to activate the database. No code changes required.

---

## Useful References

- [Supabase JavaScript client docs](https://supabase.com/docs/reference/javascript/introduction)
- [Supabase Row Level Security guide](https://supabase.com/docs/guides/auth/row-level-security)
- [Recharts API reference](https://recharts.org/en-US/api)
- [wouter routing docs](https://github.com/molefrog/wouter)
- [Azure Static Web Apps configuration reference](https://learn.microsoft.com/en-us/azure/static-web-apps/configuration)
- [Tailwind CSS v4 docs](https://tailwindcss.com/docs)
- [shields.io badge generator](https://shields.io/)
- **AI tools used:** Replit Agent — used for full scaffolding, component generation, accessibility review, code review response, and documentation generation. All output was reviewed by the developer before acceptance.
