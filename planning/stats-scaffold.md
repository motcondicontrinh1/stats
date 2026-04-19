# Feature: stats-scaffold

## Goal

Bootstrap the complete `stats` running dashboard — a Vite + React + Tailwind SPA with Strava OAuth and a Ferrari-styled UI, deployable to Vercel.

## Context

New project at `/Users/cris/Projects/stats`. GitHub remote: `https://github.com/motcondicontrinh1/stats.git`.

Strava OAuth reuses CLIENT_ID `225803` (same app as `strava-dashboard`). The token proxy pattern is identical — an Edge Function at `/api/token` keeps the client secret server-side. Source references (read-only, do not modify):

- `strava-dashboard/api/token.js` — OAuth proxy to copy verbatim
- `strava-dashboard/src/utils/auth.js` — token lifecycle to copy verbatim

Design system: Ferrari (`DESIGN.md`). Read it in full before writing any UI. Key rules:
- Absolute Black (`#000000`) for dark/hero sections; Pure White (`#FFFFFF`) for editorial sections
- Ferrari Red (`#DA291C`) only for primary CTAs — nowhere else
- FerrariSans unavailable; use **Inter** (Google Fonts) as substitute
- Body-Font labels: uppercase, 12px, 1px letter-spacing, system Arial fallback
- Border-radius: 2px for all interactive elements
- No box-shadows on cards; no gradients on UI elements

Activities filter: only show entries where `sport_type === 'Run' || type === 'Run'`.

## Steps

1. **`package.json`** — Create with name `stats`, version `1.0.0`, type `module`. Dependencies: `react@^18.3.1`, `react-dom@^18.3.1`. DevDependencies: `@vitejs/plugin-react@^4.3.1`, `vite@^5.4.10`, `tailwindcss@^3.4.14`, `autoprefixer@^10.4.20`, `postcss@^8.4.47`. Scripts: `"dev": "vite"`, `"build": "vite build"`, `"preview": "vite preview"`, `"deploy": "vercel --prod"`.

2. **`vite.config.js`** — Minimal config: import defineConfig and react plugin, return `{ plugins: [react()] }`.

3. **`tailwind.config.js`** — Extend theme with Ferrari tokens. Content: `['./index.html', './src/**/*.{js,jsx}']`. Colors to add under `theme.extend.colors`:
   - `absolute-black: '#000000'`
   - `dark-surface: '#303030'`
   - `ferrari-red: '#DA291C'`
   - `dark-red: '#B01E0A'`
   - `near-black: '#181818'`
   - `dark-gray: '#666666'`
   - `mid-gray: '#8F8F8F'`
   - `silver-gray: '#969696'`
   - `border-gray: '#CCCCCC'`
   - `button-hover: '#1EAEDB'`
   - `link-blue: '#3860BE'`
   
   FontFamily under `theme.extend.fontFamily`:
   - `sans: ['Inter', 'Arial', 'Helvetica', 'sans-serif']`
   - `label: ['Arial', 'Helvetica', 'sans-serif']`
   
   BorderRadius under `theme.extend.borderRadius`:
   - `sharp: '2px'`
   - `dialog: '8px'`

4. **`postcss.config.js`** — Export `{ plugins: { tailwindcss: {}, autoprefixer: {} } }`.

5. **`vercel.json`** — JSON with `buildCommand: "npm run build"`, `outputDirectory: "dist"`, and rewrites array: `{ source: "/api/token", destination: "/api/token.js" }` first, then `{ source: "/(.*)", destination: "/index.html" }`.

6. **`.gitignore`** — Ignore: `node_modules`, `dist`, `.env.local`, `.env`, `.vercel`.

7. **`.env.example`** — Single line: `STRAVA_CLIENT_SECRET=your_secret_here`.

8. **`index.html`** — Standard Vite HTML. In `<head>`: charset utf-8, viewport meta, title "stats", Google Fonts preconnect + link for Inter (weights 400,500,600,700). Body has `<div id="root"></div>` and `<script type="module" src="/src/main.jsx"></script>`.

9. **`api/token.js`** — Copy verbatim from `strava-dashboard/api/token.js`. No changes.

10. **`src/utils/auth.js`** — Copy verbatim from `strava-dashboard/src/utils/auth.js`. No changes.

11. **`src/utils/formatters.js`** — Create with four named exports:
    - `formatPace(metersPerSecond)`: converts m/s to "M:SS /km" string. Formula: secondsPerKm = 1000 / metersPerSecond, then format as minutes:seconds. Return `"—"` if value is falsy or <= 0.
    - `formatDistance(meters)`: returns `"X.X km"` (one decimal). Return `"—"` if falsy.
    - `formatDuration(seconds)`: returns `"Xh MMm"` if >= 3600, else `"MMm SSs"`. Return `"—"` if falsy.
    - `formatDate(isoString)`: returns short date like `"Apr 19"` using `new Date().toLocaleDateString('en-US', { month: 'short', day: 'numeric' })`.

12. **`src/index.css`** — Three Tailwind directives: `@tailwind base`, `@tailwind components`, `@tailwind utilities`. Then base styles: `body { background-color: #000000; color: #ffffff; font-family: 'Inter', Arial, Helvetica, sans-serif; }`.

13. **`src/main.jsx`** — Import React, ReactDOM createRoot, App, and `./index.css`. Mount `<App />` to `document.getElementById('root')`.

14. **`src/App.jsx`** — Adapted from `strava-dashboard/src/App.jsx`. Remove all references to Header, ActivityDrawer, CardExportModal, stats (`/athletes/{id}/stats`), selectedActivityId. Keep: all token lifecycle hooks (scheduleTokenRefresh, doRefresh, ensureValidToken), apiGet, loadDashboard, disconnect, and the init useEffect. In loadDashboard: fetch `/athlete` and `/athlete/activities?per_page=30&page=1`, then filter activities to only `sport_type === 'Run' || type === 'Run'`, set state. Screen states: `'oauth'` | `'loading'` | `'dashboard'`. JSX: render `<OAuthScreen>`, `<LoadingScreen>`, or `<Dashboard>` based on screen — no wrapper div glows, no ambient blobs, just `<div className="min-h-screen bg-absolute-black text-white font-sans">`.

15. **`src/components/OAuthScreen.jsx`** — Ferrari dark section. Full viewport height flex-col center. Structure:
    - Outer: `min-h-screen bg-absolute-black flex flex-col items-center justify-center px-6`
    - Heading: `"STATS"` — Inter, 26px (text-2xl md:text-3xl), weight 500, white
    - Subtitle label: `"RUNNING ANALYTICS"` — uppercase, font-label, 12px (text-xs), silver-gray, 1px letter-spacing (`tracking-widest`)
    - Spacer: 48px (mt-12)
    - CTA button: Ghost button style — `border border-white text-white bg-transparent px-10 py-3 rounded-sharp text-base tracking-widest font-label uppercase hover:bg-button-hover hover:border-button-hover transition-colors`. Text: `"CONNECT STRAVA"`. `href` constructed as Strava OAuth URL: `https://www.strava.com/oauth/authorize?client_id=225803&response_type=code&redirect_uri=${window.location.origin}&approval_prompt=force&scope=read,activity:read,profile:read_all`
    - Error: if `error` prop, show it below in ferrari-red text, text-sm, font-label uppercase, mt-6

16. **`src/components/LoadingScreen.jsx`** — Props: `text`. Absolute Black full-height centered layout. Display `text` prop in white, font-label uppercase, 12px, 1px letter-spacing, silver-gray. Below it, a blinking ellipsis: three dots using CSS `animate-pulse`. Keep it minimal.

17. **`src/components/Dashboard.jsx`** — Props: `profile`, `activities`, `onDisconnect`. Two-section Ferrari chiaroscuro layout:

    **Section 1 — Dark hero (bg-absolute-black, py-16 px-6 md:px-12)**:
    - Athlete name: `{profile.firstname} {profile.lastname}` — Inter 26px/500, white
    - Stats row below (mt-4 flex gap-8): three stat blocks. Each block: value in white Inter 16px/700, label in silver-gray font-label uppercase text-xs tracking-widest. Stats to display: total runs (activities.length), total distance (`formatDistance(sum of distances)`), avg pace (`formatPace(average of average_speed values)`). Use formatters from `src/utils/formatters.js`.
    - Disconnect: `<button onClick={onDisconnect}>` — white text, underline, font-label uppercase text-xs tracking-widest, mt-8 block

    **Section 2 — White editorial list (bg-white py-12 px-6 md:px-12)**:
    - Section label: `"RECENT RUNS"` — font-label uppercase text-xs tracking-widest text-mid-gray mb-8
    - Runs list: `<ul>` with dividers (`divide-y divide-border-gray`). Each `<li>` is a run row (`py-5 flex items-baseline gap-6`):
      - Date: `formatDate(run.start_date_local)` — font-label uppercase text-xs tracking-widest text-silver-gray, w-16
      - Run name: `run.name` — Inter text-base/700, text-near-black, flex-1 truncate
      - Distance: `formatDistance(run.distance)` — font-label text-xs text-dark-gray uppercase tracking-widest
      - Pace: `formatPace(run.average_speed)` — font-label text-xs text-dark-gray uppercase tracking-widest
      - Duration: `formatDuration(run.moving_time)` — font-label text-xs text-dark-gray uppercase tracking-widest
    - If activities array is empty: show `"NO RUNS FOUND"` in font-label uppercase mid-gray centered

## File Targets

- `package.json` — create
- `vite.config.js` — create
- `tailwind.config.js` — create
- `postcss.config.js` — create
- `vercel.json` — create
- `.gitignore` — create
- `.env.example` — create
- `index.html` — create
- `api/token.js` — create (copy from strava-dashboard)
- `src/utils/auth.js` — create (copy from strava-dashboard)
- `src/utils/formatters.js` — create
- `src/index.css` — create
- `src/main.jsx` — create
- `src/App.jsx` — create
- `src/components/OAuthScreen.jsx` — create
- `src/components/LoadingScreen.jsx` — create
- `src/components/Dashboard.jsx` — create

## Acceptance Criteria

- [ ] `npm install` completes without errors
- [ ] `npm run dev` starts Vite dev server on localhost:5173
- [ ] OAuth screen renders on Absolute Black with "STATS" heading, "RUNNING ANALYTICS" label, and Ghost Button — matching Ferrari design system
- [ ] "CONNECT STRAVA" button redirects to Strava OAuth authorization page
- [ ] After authorizing, only activities with `sport_type === 'Run' || type === 'Run'` appear — no rides, walks, or other types
- [ ] Stats row (total runs, total distance, avg pace) computes correctly from the filtered runs array
- [ ] Chiaroscuro layout renders: dark hero section above, white runs list below
- [ ] Ferrari Red (`#DA291C`) does not appear anywhere in the initial scaffold (no CTAs use red yet — Ghost Button on dark bg is appropriate for connect)
- [ ] `npm run build` produces a clean `dist/` without TypeScript or Vite errors
- [ ] No console errors on initial page load
