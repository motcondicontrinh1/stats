# Feature: stats-scaffold

## Goal

Bootstrap the complete `stats` running dashboard — a Vite + React + Tailwind SPA with Strava OAuth, deployable to Vercel. The UI should feel genuinely surprising and fresh — athletic, data-forward, unlike anything a typical running app looks like. Use the Ferrari design tokens as raw material but don't imitate Ferrari's website layout.

## Context

New project at `/Users/cris/Projects/stats`. GitHub remote: `https://github.com/motcondicontrinh1/stats.git`.

Strava OAuth uses CLIENT_ID `225803`. An Edge Function at `/api/token` keeps the client secret server-side. 
Design system: Ferrari (`DESIGN.md`). Read it in full before writing any UI.

**The goal is NOT to reproduce Ferrari's website.** The goal is to use Ferrari's token system — its colors, its typographic precision, its restraint — to build something that feels unexpected for a running dashboard. Think: a personal timing board, a race result screen, a data instrument. Something a runner would open every morning and feel proud of. Surprise the user.

Hard constraints from DESIGN.md:
- Ferrari Red (`#DA291C`) only for one primary CTA — nowhere else
- FerrariSans unavailable; use **Inter** from Google Fonts as substitute
- Body-Font (uppercase labels, 1px letter-spacing): use Arial/Helvetica
- Border-radius: 2px for interactive elements, no pill shapes
- No box-shadows on content; no UI gradients

Everything else — layout, rhythm, which sections are dark vs light, how data is displayed, typographic hierarchy — is the implementer's creative call. Make it remarkable.

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

9. **`api/token.js`** — Vercel Edge Function (`export const config = { runtime: 'edge' }`). Accepts POST with JSON body. Handles two grant types: `authorization_code` (requires `code` + `redirect_uri`) and `refresh_token` (requires `refresh_token`). Posts to `https://www.strava.com/oauth/token` with `client_id: '225803'` and `client_secret: process.env.STRAVA_CLIENT_SECRET`. Returns `{ access_token, refresh_token, expires_at, athlete }`. Include CORS headers and OPTIONS preflight support.

10. **`src/utils/auth.js`** — Token lifecycle utilities using localStorage. Keys: `stats_access_token`, `stats_refresh_token`, `stats_expires_at`. Named exports: `saveTokens(tokenData)`, `loadTokens()` → `{ accessToken, refreshToken, expiresAt }`, `clearTokens()`, `isTokenExpired()`, `needsRefresh()` (true if within 300s of expiry), `exchangeTokenViaProxy(code)` (POST `/api/token` with code + redirect_uri), `refreshTokenViaProxy(refreshToken)` (POST `/api/token` with refresh_token grant). Export `REFRESH_BUFFER = 300`.

11. **`src/utils/formatters.js`** — Create with four named exports:
    - `formatPace(metersPerSecond)`: converts m/s to "M:SS /km" string. Formula: secondsPerKm = 1000 / metersPerSecond, then format as minutes:seconds. Return `"—"` if value is falsy or <= 0.
    - `formatDistance(meters)`: returns `"X.X km"` (one decimal). Return `"—"` if falsy.
    - `formatDuration(seconds)`: returns `"Xh MMm"` if >= 3600, else `"MMm SSs"`. Return `"—"` if falsy.
    - `formatDate(isoString)`: returns short date like `"Apr 19"` using `new Date().toLocaleDateString('en-US', { month: 'short', day: 'numeric' })`.

12. **`src/index.css`** — Three Tailwind directives: `@tailwind base`, `@tailwind components`, `@tailwind utilities`. Then base styles: `body { background-color: #000000; color: #ffffff; font-family: 'Inter', Arial, Helvetica, sans-serif; }`.

13. **`src/main.jsx`** — Import React, ReactDOM createRoot, App, and `./index.css`. Mount `<App />` to `document.getElementById('root')`.

14. **`src/App.jsx`** — Root component managing auth state and screen routing. Use React hooks (useState, useEffect, useRef, useCallback). Token lifecycle: `scheduleTokenRefresh`, `doRefresh`, `ensureValidToken` using utils from `src/utils/auth.js`. `apiGet(endpoint)` wraps fetch to `https://www.strava.com/api/v3` with Bearer token, auto-refreshes on 401. `loadDashboard` fetches `/athlete` then `/athlete/activities?per_page=30&page=1`, filters to `sport_type === 'Run' || type === 'Run'`, sets state. `disconnect` clears tokens and resets to oauth screen. Init useEffect handles OAuth callback (code in query string) and restores existing sessions. Screen states: `'oauth'` | `'loading'` | `'dashboard'`. Renders `<OAuthScreen>`, `<LoadingScreen>`, or `<Dashboard>` accordingly.

15. **`src/components/OAuthScreen.jsx`** — The entry point. Needs to feel intentional, not like a placeholder. Props: `error`. Must include an anchor tag that navigates to the Strava OAuth URL: `https://www.strava.com/oauth/authorize?client_id=225803&response_type=code&redirect_uri=${window.location.origin}&approval_prompt=force&scope=read,activity:read,profile:read_all`. If `error` prop is present, display it. Design is the implementer's creative call — use the Ferrari token system from `DESIGN.md`.

16. **`src/components/LoadingScreen.jsx`** — Props: `text`. Shows `text` while data loads. Keep it minimal and on-brand.

17. **`src/components/Dashboard.jsx`** — Props: `profile`, `activities`, `onDisconnect`. Displays the athlete's name, aggregate stats (total runs, total distance, avg pace computed from the activities array using formatters), and the filtered run list (each row: date, name, distance, pace, duration). Include a disconnect trigger. The visual design — how these pieces are arranged, what's large, what's small, the rhythm — is the implementer's creative call. Make it feel like a personal instrument, not a data table. Use formatters from `src/utils/formatters.js`.

## File Targets

- `package.json` — create
- `vite.config.js` — create
- `tailwind.config.js` — create
- `postcss.config.js` — create
- `vercel.json` — create
- `.gitignore` — create
- `.env.example` — create
- `index.html` — create
- `api/token.js` — create
- `src/utils/auth.js` — create
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
- [ ] OAuth screen renders and the Strava OAuth link is correct
- [ ] After authorizing, only `sport_type === 'Run' || type === 'Run'` activities appear — no rides, walks, or other types
- [ ] Aggregate stats (total runs, total distance, avg pace) compute correctly from the filtered array
- [ ] Ferrari Red (`#DA291C`) appears at most once — on the primary CTA only
- [ ] Border-radius on interactive elements is 2px, not rounded/pill
- [ ] `npm run build` produces a clean `dist/` without errors
- [ ] No console errors on initial page load
- [ ] `npm run deploy` succeeds and the app is live on Vercel
