# Food Preferences Survey

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

## Description

Food Preferences Survey is a web application that collects food preference data from college students through a short, four-question form and displays aggregated results as interactive bar charts. It was built for BAIS:3300 (Spring 2026) to demonstrate full-stack data collection using a modern React front end backed by a Supabase PostgreSQL database. The app is designed for students and instructors who want a simple, shareable survey with live results — no login required.

## Features

- Take a four-question survey about food habits in under a minute
- Client-side validation with clear, accessible error messages on every field
- Responses are saved instantly to a Supabase database with no page reload
- View aggregated results on a dedicated Results page — no account needed
- Three bar charts break down takeout frequency, enjoyed foods, and most common meal
- "Other" food entries are normalized and counted alongside standard options
- Fully keyboard-navigable form with ARIA labels and error descriptions
- Deployable to both Replit and Azure Static Web Apps with no code changes

## Tech Stack

| Technology | Purpose |
|---|---|
| React 18 | UI component framework |
| TypeScript | Static typing across all source files |
| Vite | Dev server and production bundler |
| Tailwind CSS | Utility-first styling |
| shadcn/ui | Accessible UI primitives (buttons, inputs) |
| wouter | Lightweight client-side routing |
| Recharts | SVG bar charts on the Results page |
| Supabase | Hosted PostgreSQL database + REST API |
| pnpm workspaces | Monorepo package management |

## Getting Started

### Prerequisites

- [Node.js 18+](https://nodejs.org/)
- [pnpm 8+](https://pnpm.io/installation)
- A [Supabase](https://supabase.com/) project (free tier works)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/food-survey.git
   cd food-survey
   ```

2. Install dependencies:
   ```bash
   pnpm install
   ```

3. Create the database table by running the following SQL in your **Supabase dashboard → SQL Editor**:
   ```sql
   CREATE TABLE survey_responses (
     id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
     created_at timestamptz DEFAULT now(),
     favorite_food text NOT NULL,
     common_meal text NOT NULL,
     takeout_frequency text NOT NULL,
     enjoyed_foods text[] NOT NULL,
     other_food text
   );

   ALTER TABLE survey_responses ENABLE ROW LEVEL SECURITY;

   CREATE POLICY "Allow public insert"
     ON survey_responses FOR INSERT WITH CHECK (true);

   CREATE POLICY "Allow public select"
     ON survey_responses FOR SELECT USING (true);
   ```

4. Create a `.env` file in `artifacts/food-survey/` with your Supabase credentials:
   ```env
   VITE_SUPABASE_URL=https://your-project.supabase.co
   VITE_SUPABASE_ANON_KEY=your-anon-key
   ```

5. Start the development server:
   ```bash
   pnpm --filter @workspace/food-survey run dev
   ```

6. Open your browser to the URL printed in the terminal (typically `http://localhost:5173`).

## Usage

- Navigate to `/` for the home page. Click **Take the Survey** to begin.
- Complete all four questions and click **Submit Response**.
- After submission, a disabled **Response Submitted** button confirms your entry. Click **View Results** to see the charts.
- Navigate to `/results` at any time to view live aggregated responses from all participants.

### Configuration

| Environment Variable | Description |
|---|---|
| `VITE_SUPABASE_URL` | Your Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | Your Supabase publishable (anon) key |

Both variables are `VITE_` prefixed, meaning Vite bundles them into the client JavaScript at build time. The Supabase anon key is intentionally public-facing and is safe for browser use when Row Level Security policies are in place.

### Deploying to Azure Static Web Apps

A `staticwebapp.config.json` is included in `artifacts/food-survey/` for SPA routing. When creating the Static Web App in Azure, use these build settings:

| Setting | Value |
|---|---|
| `app_location` | `artifacts/food-survey` |
| `api_location` | *(leave empty)* |
| `output_location` | `dist` |

Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` under **Azure Portal → Static Web App → Settings → Environment variables**, then trigger a new deployment.

## Project Structure

```
artifacts/food-survey/
├── public/                        # Static assets served as-is
├── src/
│   ├── pages/
│   │   ├── Home.tsx               # Landing page with CTA buttons
│   │   ├── Survey.tsx             # Four-question form + thank-you screen
│   │   └── Results.tsx            # Recharts bar charts + total count
│   ├── components/
│   │   └── Footer.tsx             # Shared footer displayed on all pages
│   ├── lib/
│   │   └── supabase.ts            # Supabase client initialized from env vars
│   ├── App.tsx                    # wouter router — defines /, /survey, /results
│   └── index.css                  # Tailwind base + CSS custom properties
├── staticwebapp.config.json       # Azure SWA SPA routing fallback config
├── vite.config.ts                 # Vite dev server and build configuration
└── package.json                   # Package metadata and scripts
```

## Changelog

### v1.0.0 — 2026-03-26

- Initial release
- Home, Survey, and Results pages with full routing
- Four-question survey form with client-side validation and ARIA accessibility
- Supabase integration for response persistence
- Recharts bar charts for takeout frequency, enjoyed foods, and meal type
- "Other" food normalization and aggregation
- Azure Static Web Apps routing config included
- MIT License

## Known Issues / To-Do

- [ ] No server-side deduplication — the same user can submit multiple responses
- [ ] Results page fetches all rows on every load; large datasets may cause slow renders
- [ ] "Other" food free-text is case-folded but not spell-checked, so typos create separate chart entries

## Roadmap

- Add a unique submission token (stored in `localStorage`) to prevent duplicate responses
- Paginate or stream results on the Results page for large datasets
- Add a simple admin view protected by Supabase Auth to view and export raw responses as CSV
- Support additional question types (Likert scale, ranked choice) for future survey iterations
- Add animated chart transitions on the Results page using Recharts' built-in animation props

## Contributing

Contributions, bug reports, and suggestions are welcome. Please open an issue first to discuss any significant changes before submitting a pull request.

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m "feat: describe your change"`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a pull request against `main` and describe what you changed and why

## License

This project is licensed under the [MIT License](LICENSE).

## Author

**Angela**
University of Iowa — Tippie College of Business
BAIS:3300 · Spring 2026

## Contact

GitHub: [github.com/your-username](https://github.com/your-username)

## Acknowledgements

- [Supabase Docs](https://supabase.com/docs) — database setup and Row Level Security guidance
- [Recharts](https://recharts.org/) — charting library and API reference
- [shadcn/ui](https://ui.shadcn.com/) — accessible component primitives
- [Tailwind CSS](https://tailwindcss.com/docs) — utility-first styling documentation
- [Vite](https://vitejs.dev/) — build tool and dev server
- [wouter](https://github.com/molefrog/wouter) — lightweight React routing
- [shields.io](https://shields.io/) — README badge generation
- [Replit](https://replit.com/) — development and hosting environment
- AI assistant (Replit Agent) — scaffolding, component generation, and accessibility review
