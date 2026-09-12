# ChartSense: AI Data-to-Chart Web App
## Build prompts, page by page and function by function

Paste the prompts into your AI coding tool (Claude Code, Cursor, Lovable, Bolt, v0, etc.) **in order**. Wait for each one to finish, run the app, and fix anything broken before you paste the next one.
Prompt 0 is the master context. Paste it first, and paste it again whenever you start a new chat.

**The 3D approach:** Every page has its own 3D moment. It all comes from **one shared 3D component library** (Phase 1B) with a built-in performance guard. That's what keeps the effects smooth: each page reuses tested components instead of building new 3D code from scratch, and every scene automatically scales down on weaker devices. Build Phase 1B before any page.

---

## PROMPT 0: Master Context (paste first, every session)

```
You are a senior full-stack engineer. We are building "ChartSense", a production-ready web app where:
1. Users sign up / log in.
2. Users upload Excel (.xlsx, .xls), CSV, TSV, JSON, or Parquet files (optionally several files + a free-text description of the data).
3. The system parses the file, profiles every column, DETECTS DATA ERRORS, and shows a validation report.
4. The system recommends and renders the best charts from a catalogue of 45 chart types, based on the data schema.
5. An AI layer (Claude API) explains the data, justifies each chart choice, and writes insights.

TECH STACK (do not deviate unless I say so):
- Frontend: Next.js 14+ (App Router) + TypeScript + Tailwind CSS + shadcn/ui + lucide-react icons
- Charts: Apache ECharts (echarts-for-react) as the main library, plus echarts-gl (3D charts), echarts-wordcloud, d3-chord for chord diagrams, and ECharts geo maps for choropleths
- 3D and motion: three.js + @react-three/fiber + @react-three/drei + @react-three/postprocessing (bloom, depth of field, chromatic aberration used sparingly), framer-motion (UI motion, 3D tilt, page transitions), GSAP + ScrollTrigger (scroll-driven scenes), Lenis (smooth scrolling), detect-gpu (device tiering), maath (easing/random helpers)
- Auth: Auth.js (NextAuth v5) with Credentials (email + password, bcrypt) and Google OAuth
- Database: PostgreSQL with the Prisma ORM
- File storage: S3-compatible storage (local MinIO in dev)
- Analysis service: Python 3.11 FastAPI microservice using pandas, numpy, scipy, openpyxl, pyarrow, and scikit-learn (for clustering and dendrograms)
- AI: Anthropic Claude API (model "claude-sonnet-5"), called ONLY from the server
- Background jobs: BullMQ + Redis (parsing and analysis run async)
- Validation: zod (TS) and pydantic (Python)

GOLDEN RULES:
- The AI NEVER invents numbers. All statistics and chart data are computed deterministically in Python. The AI receives only the schema, the summary stats, and a small sample, and returns chart choices plus explanations as strict JSON.
- Every API route checks the session and makes sure the user owns the resource.
- Never send raw full datasets to the AI. Send at most 50 sample rows.
- All code must be typed, modular, and commented at function level. Put reusable logic in /lib (TS) or /app/services (Python).
- The UI must be responsive, support dark and light mode, and show loading, empty, and error states on every page.

DESIGN LANGUAGE ("Premium 3D Data Studio"):
- A premium, cinematic, professional look (think Linear, Stripe, Vercel, Apple product pages). Never cartoonish, never cluttered.
- Dark-first theme: background #05070F → #0B1024 gradient; glass panels (backdrop-blur 20px, white 6% fill, 1px white 10% border, inner highlight); accent gradient indigo #6366F1 → cyan #22D3EE → violet #A855F7; success #10B981, warning #F59E0B, error #F43F5E. The light theme mirrors it with frosted-white glass.
- Typography: "Geist" or "Inter" for UI, "Space Grotesk" for display headings, and tabular numbers for all figures.
- Depth everywhere: every page has at least ONE signature WebGL 3D element plus CSS 3D micro-interactions (tilt cards, layered parallax, magnetic buttons, depth shadows, glowing borders that follow the cursor).
- Motion rules: ease [0.22, 1, 0.36, 1]; UI transitions 200-400ms; 3D camera moves 800-1500ms; stagger lists by 40ms. Motion must feel smooth and physical, never jittery.

3D PERFORMANCE RULES (non-negotiable; the effects must work perfectly):
- Use the shared components from /web/components/3d ONLY. Never create an ad-hoc <Canvas>.
- ONE global WebGL <Canvas> per page, using drei <View> to render several 3D areas through it (browsers cap WebGL contexts at about 16, and multiple canvases cause crashes and flicker).
- Load every 3D scene with next/dynamic({ ssr:false }) + <Suspense> and a static poster-image fallback. Text and CTAs render as HTML first; 3D must NEVER block First Contentful Paint (LCP) or interaction.
- Device tiering via detect-gpu: HIGH = full effects + postprocessing; MEDIUM = no postprocessing, fewer particles; LOW / mobile / battery-saver = simplified scene; NO WEBGL = CSS-3D fallback with the same layout.
- drei <PerformanceMonitor> + <AdaptiveDpr> + <AdaptiveEvents>: automatically lower DPR and particle count if FPS drops below 50. DPR is capped at [1, 2].
- Pause rendering when a scene is off-screen (IntersectionObserver) or the tab is hidden; use frameloop="demand" for static scenes.
- Respect prefers-reduced-motion: disable auto-rotation, parallax, and scroll scenes, and show still frames instead.
- Use instancing for anything repeated more than 50 times (InstancedMesh), compressed assets (Draco/Meshopt geometry, KTX2 textures), and dispose geometries and materials on unmount.
- Budgets: 60 fps on a mid-range laptop, ≥ 30 fps on a mid-range phone; each page's 3D bundle ≤ 250KB gzipped (lazy-loaded); Lighthouse Performance ≥ 85 and Accessibility ≥ 95.
- Every 3D element is decorative or duplicated in accessible HTML (aria-hidden on canvases; data is always available as a table or text).

FOLDER STRUCTURE:
/web          -> Next.js app
/analysis     -> FastAPI service
/docker-compose.yml -> postgres, redis, minio, analysis, web

Confirm you understand, then wait for the next prompt.
```

---

## PHASE 1: Project Setup

### Prompt 1.1: Scaffold
```
Create the monorepo from the Master Context:
- /web: Next.js 14 App Router, TypeScript, Tailwind, shadcn/ui initialised (button, input, card, dialog, table, tabs, toast, dropdown-menu, badge, progress, skeleton, tooltip, sheet, select, switch), ESLint + Prettier.
- /analysis: FastAPI project with /app/main.py, /app/routers, /app/services, /app/models, requirements.txt, pytest set up.
- docker-compose.yml with postgres:16, redis:7, minio, analysis (port 8000), web (port 3000).
- .env.example listing: DATABASE_URL, AUTH_SECRET, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, S3_ENDPOINT, S3_BUCKET, S3_ACCESS_KEY, S3_SECRET_KEY, REDIS_URL, ANALYSIS_SERVICE_URL, ANALYSIS_SERVICE_TOKEN, ANTHROPIC_API_KEY, MAX_UPLOAD_MB=50.
- A README with setup steps.
The analysis service must only accept requests carrying the header X-Service-Token = ANALYSIS_SERVICE_TOKEN.
```

### Prompt 1.2: Database Schema (Prisma)
```
Create prisma/schema.prisma with these models:
- User: id, name, email (unique), passwordHash (nullable for OAuth), emailVerified, image, role (USER|ADMIN), plan (FREE|PRO), createdAt
- Account, Session, VerificationToken (standard Auth.js adapter models)
- PasswordResetToken: id, userId, tokenHash, expiresAt, usedAt
- Project: id, userId, name, description, createdAt, updatedAt   (a project groups several uploads)
- Dataset: id, projectId, userId, originalFileName, fileType, sizeBytes, storageKey, sheetNames (Json), selectedSheet, status (UPLOADED|PARSING|VALIDATING|ANALYZING|READY|FAILED), rowCount, columnCount, userContext (text: the user's description of the data), errorMessage, createdAt
- ColumnProfile: id, datasetId, name, originalName, index, detectedType (NUMERIC|INTEGER|CATEGORICAL|BOOLEAN|DATETIME|TEXT|ID|GEO_COUNTRY|GEO_STATE|LAT|LON|CURRENCY|PERCENT), semanticRole (METRIC|DIMENSION|TIME|IDENTIFIER|GEO|TEXT|OHLC_OPEN|OHLC_HIGH|OHLC_LOW|OHLC_CLOSE|START_DATE|END_DATE|SOURCE|TARGET|PARENT), nullCount, uniqueCount, stats (Json), sampleValues (Json)
- ValidationIssue: id, datasetId, severity (ERROR|WARNING|INFO), code, column (nullable), rowIndexes (Json, capped at 100), affectedCount, message, suggestion, autoFixable (bool), fixed (bool)
- ChartRecommendation: id, datasetId, chartType (enum of all 45 types), family, title, config (Json: which columns map to x/y/size/color/etc.), chartData (Json: precomputed series), score (0-100), aiReason (text), aiInsight (text), pinned (bool), order
- Report: id, userId, projectId, title, shareToken (nullable, unique), isPublic, chartIds (Json), createdAt
- AuditLog: id, userId, action, meta (Json), createdAt
Add indexes on userId and datasetId. Generate the migration and a seed script with one demo user.
```

---

## PHASE 1B: 3D Design System (build this BEFORE any page)

### Prompt 1B.1: Design Tokens and Theme
```
Set up the "Premium 3D Data Studio" design system from the Master Context:
- Tailwind theme tokens + CSS variables for both themes: background layers (bg-0, bg-1, bg-2), glass surfaces, borders, text (primary/secondary/muted), accent gradient, status colours, and chart palette (8 colour-blind-safe colours that glow well on dark).
- Utilities: .glass, .glass-strong, .gradient-border (animated conic-gradient border), .glow-{color}, .noise-overlay (subtle SVG grain), .text-gradient, and depth shadows (shadow-depth-1..4, layered and soft).
- Fonts via next/font: Geist/Inter (UI), Space Grotesk (display), JetBrains Mono (numbers/code).
- A <CursorSpotlight /> component: a radial light that follows the cursor over glass cards (CSS variables updated with rAF; no React re-renders).
- Restyle the shadcn/ui components (Button, Input, Card, Dialog, Tabs, Table, Toast, Select, Badge) so they match: glass, glowing focus rings, and smooth press/hover states.
```

### Prompt 1B.2: 3D Engine Core (/web/components/3d/core)
```
Build the 3D infrastructure that every page uses. Each item is its own file, typed and documented:

- useDeviceTier(): uses detect-gpu + navigator.hardwareConcurrency + deviceMemory + saveData + prefers-reduced-motion + the user's "3D quality" setting. Returns 'HIGH' | 'MEDIUM' | 'LOW' | 'OFF'. Cache the result in localStorage (wrapped in try/catch).
- <Scene3DProvider>: mounted once in the root layout. Renders ONE fixed, full-screen <Canvas> behind the content (eventSource = document.body, eventPrefix "client") with drei <View.Port />. Includes <PerformanceMonitor onDecline → lower tier>, <AdaptiveDpr pixelated={false}/>, <AdaptiveEvents/>, and dpr={[1, tier==='HIGH' ? 2 : 1.5]}. Also makes the tier available through context.
- <View3D fallback={<img .../>} className>: a div-tracked drei <View>. It pauses (frameloop off) when off-screen via IntersectionObserver or when document.hidden, and renders the poster fallback when tier === 'OFF' or WebGL is unavailable.
- <QualityGate min="MEDIUM">: renders its children only at or above that tier.
- <Effects>: postprocessing (Bloom with a luminance threshold, subtle Vignette, and optional DepthOfField) at HIGH tier only.
- useWebGLContextGuard(): listens for webglcontextlost/restored; on a loss it shows the fallbacks and tries to restore without a page reload.
- useScrollProgress(ref): Lenis smooth scroll synced with GSAP ScrollTrigger, returning 0-1 progress for scroll-driven scenes.
- usePointerParallax(strength): a smoothed (damped) pointer position for camera/object parallax.
- disposeObject(obj): recursively disposes geometries, materials, and textures.
- A /dev/3d playground page: every 3D component on a grid, an r3f-perf FPS overlay (dev only), and a tier switcher to test HIGH/MEDIUM/LOW/OFF.
```

### Prompt 1B.3: Reusable 3D Component Library (/web/components/3d)
```
Build these components. Each must accept `tier` behaviour, `reducedMotion`, and colour props, and must ship with a static poster fallback (render a PNG of each one with a script and save it to /public/posters):

1. <ParticleMorph shapes={['grid','bars','sphere','globe']} progress>: a GPU particle system (custom shader on THREE.Points; 40k particles on HIGH, 15k on MEDIUM, 5k on LOW). It morphs between target shapes by interpolating attributes in the vertex shader. Particles glow along the accent gradient and react to the cursor with a gentle repulsion.
2. <DataGlobe points arcs>: a globe with an atmosphere glow (fresnel shader), instanced data points, and animated arcs (tube + dash-offset shader). It rotates slowly and can be dragged (inertia).
3. <FloatingChartShapes>: glass 3D icons of a bar chart, a pie slice, a line ribbon, and a scatter cluster, floating with drei <Float>. HIGH: MeshTransmissionMaterial; MEDIUM/LOW: MeshPhysicalMaterial (much cheaper).
4. <HoloBarChart3D data>: holographic bars that rise from a glowing grid floor with a spring animation; hover lifts a bar and shows an HTML tooltip via drei <Html>.
5. <DataCrystal state="idle|processing|success|error" progress>: a faceted crystal that stands for a dataset. It rotates, pulses while processing, turns green and bursts particles on success, and cracks red on error.
6. <PortalRing active intensity>: a vortex ring shader (noise + polar UVs) that spins faster and glows brighter when active. It's the upload drop target.
7. <Gauge3D value max>: a 3D ring gauge with a glowing progress arc that animates from its old to its new value, coloured red→amber→green.
8. <IssueTerrain cells>: an InstancedMesh grid of columns × row-buckets whose height/colour encode issue density. It supports hover and click, and "heals" cells (red→green ripple) when an issue is fixed.
9. <NetworkConstellation nodes links>: glowing nodes and animated line links with a force layout (d3-force-3d computed in a web worker).
10. <AIOrb state="idle|thinking|speaking">: a shader sphere with noise displacement whose amplitude follows the streaming token rate.
11. <GradientMeshBackground intensity>: a cheap full-screen animated noise gradient (one fragment shader) used as the ambient app background.
12. <Carousel3D items>: a cylinder ring of HTML/3D cards; drag or scroll to rotate, with snap and inertia.

CSS/HTML 3D components (framer-motion, no WebGL, so they are safe on every device):
13. <TiltCard>: perspective 1000px; rotateX/Y up to 8° following the cursor (spring); a glare highlight layer; children lifted with translateZ; resets smoothly on leave.
14. <MagneticButton>: moves slightly toward the cursor, with a glow ripple on click.
15. <FlipNumber value>: 3D flip-digit counter for KPIs.
16. <DepthParallax layers>: layered elements that move at different depths with the pointer and scroll.
17. <Loader3D>: a branded loading animation (rotating wireframe cube becoming a bar chart) with a CSS-only fallback.
```

### Prompt 1B.4: Motion and Page Transitions
```
- app/(app)/template.tsx and app/(marketing)/template.tsx: framer-motion page transitions (fade + 12px rise + slight scale 0.98 → 1, 350ms). A shared-element transition moves a dataset card into its detail page header (layoutId).
- The 3D camera eases to a new preset per route (defined in /lib/3d/cameraPresets.ts) instead of cutting.
- useReveal(): scroll-reveal hook (stagger children and blur-in) with reduced-motion support.
- Lenis smooth scroll on marketing pages only (not on data tables).
- All motion goes through /lib/motion/tokens.ts (durations, easings, springs) so the feel is consistent.
```

---

## PHASE 2: Authentication Pages

### Prompt 2.1: Auth Config and Functions
```
Implement Auth.js v5 in /web/auth.ts:
- Providers: Credentials (email + password) and Google.
- Prisma adapter, JWT session strategy, and session.user includes id, role, plan.

Create /web/lib/auth/ with these functions (each one exported, typed, and unit-tested):
- hashPassword(plain): Promise<string>            -> bcrypt, cost 12
- verifyPassword(plain, hash): Promise<boolean>
- validatePasswordStrength(pw): {valid, errors[]}  -> min 8 chars, upper, lower, number, symbol
- createUser({name,email,password})                -> rejects duplicate email with a clear error
- generateVerificationToken(email) / verifyEmailToken(token)
- generatePasswordResetToken(email)                -> stores a SHA-256 hash, expires in 1 hour
- resetPassword(token, newPassword)
- requireUser()                                    -> server helper that returns the session user or throws/redirects to /login
- rateLimit(key, limit, windowSec)                 -> Redis-based; use it on login, signup, and reset (5 attempts per 15 min)

Add middleware.ts protecting /dashboard, /upload, /datasets, /projects, /reports, /settings. Redirect unauthenticated users to /login?callbackUrl=...
Send email through a pluggable sendEmail() function (Resend or SMTP via nodemailer; log to the console in dev).
```

### Prompt 2.2: Page: Landing (`/`)
```
Build the public landing page at app/(marketing)/page.tsx. It is the "wow" page and must feel like a premium product launch. Use ONLY the Phase 1B components.

1. NAVBAR: a floating glass pill navbar that shrinks and blurs more on scroll; links (Features, Chart Gallery, Pricing, Docs); Login + a MagneticButton "Get Started" with an animated gradient border.

2. HERO (full viewport):
   - Left: headline "Upload your spreadsheet. Get the right charts instantly." with a split-text reveal (words rise in with blur → sharp, 40ms stagger), subtext, two CTAs, and a trust line ("45 chart types · 30 error checks · AI insights").
   - Background/right: <ParticleMorph> telling the product story on a loop: particles form a flat spreadsheet GRID → some cells turn RED (errors) → a scanning light beam sweeps across and the red cells turn cyan (fixed) → particles fly up into a 3D BAR CHART → then morph into a GLOBE → loop. Pointer parallax on the camera, bloom on HIGH tier.
   - The HTML headline and a static poster render instantly; the 3D fades in after load.

3. SCROLL STORY (pinned section, GSAP ScrollTrigger, 3 stages driven by scroll progress):
   - Stage 1 "Upload": a 3D spreadsheet sheet drops into a <PortalRing>.
   - Stage 2 "Detect": the sheet unfolds into an <IssueTerrain>; red spikes are errors with floating labels ("#DIV/0! in D14", "12 duplicate rows", "'USA' vs 'usa'"), and then they heal to green.
   - Stage 3 "Visualize": the terrain reshapes into a <HoloBarChart3D>, then line and pie charts bloom around it.
   - Side text for each stage fades in sync. Reduced motion: 3 static illustrated cards.

4. CHART FAMILIES: 7 <TiltCard>s (glass, cursor spotlight), each with a small looping 3D mini-chart of that family rendered through the shared canvas via <View3D>. Hovering lifts the card and speeds up its animation.

5. CHART GALLERY PREVIEW: a <Carousel3D> ring of 45 chart thumbnails; drag to spin; clicking opens /gallery.

6. LIVE DEMO: a "Try it now" glass panel. The user drops or picks a sample file and a mini in-browser demo runs (mocked pipeline with a <DataCrystal> going processing → success) and reveals 3 real charts. No login needed.

7. STATS: big <FlipNumber> counters (charts generated, errors caught, files analyzed) over <DepthParallax> floating shapes.

8. TESTIMONIALS: two marquee rows moving in opposite directions with depth blur on the edges.

9. PRICING: 3 <TiltCard> plans; PRO is raised with a rotating gradient border glow and a "Most popular" badge; monthly/yearly toggle animates the price digits.

10. FAQ accordion (smooth height animation), then a FINAL CTA over a slowly rotating <DataGlobe> with data arcs, then the footer.

Fully responsive: on mobile the hero uses the LOW-tier particle count and the scroll story becomes a vertical swipe of 3 cards. Dark and light mode.
```

### Prompt 2.3: Page: Sign Up (`/signup`)
```
Build app/(auth)/signup/page.tsx:
- Fields: Full name, Email, Password (with a live strength meter using validatePasswordStrength), Confirm password, and an "I agree to Terms" checkbox.
- "Continue with Google" button.
- Client validation with react-hook-form + zod; server action signUpAction() calls createUser(), sends the verification email, and redirects to /verify-email?email=...
- Show field-level errors, a loading spinner on submit, and a toast on server error (e.g. "Email already registered").
- Link to /login.

3D DESIGN (shared by all auth pages; build it once as <AuthShell>):
- Split screen: on the left the form sits in a strong-glass card with a gradient border; on the right a full-height 3D scene with a <DataCrystal> surrounded by <FloatingChartShapes> orbiting it.
- The scene reacts to the form: each keystroke sends a soft particle pulse through the crystal; the password strength meter drives the crystal's colour (red → amber → green) and brightness; a focused input tilts the camera slightly toward the form.
- On submit: the crystal spins up (processing). On success: the shapes assemble into a glowing check mark, then a page transition to the next screen. On error: a small shake of the card and a brief red crack effect on the crystal.
- The AuthShell scene stays mounted between /signup, /login, /forgot-password, and /reset-password, so switching pages only moves the camera (no reload flash).
- Mobile: the 3D scene becomes a short animated header band above the form.
```

### Prompt 2.4: Page: Login (`/login`)
```
Build app/(auth)/login/page.tsx:
- Email, Password (show/hide toggle), a "Remember me" checkbox, a "Forgot password?" link, and a Google button.
- Server action loginAction(): rate-limited; generic error message "Invalid email or password" (never reveal which one is wrong).
- If the email is not verified, show a "Resend verification email" button.
- On success, redirect to callbackUrl or /dashboard.
- Use <AuthShell>. The camera preset is "front". The success animation zooms the camera INTO the crystal, which dissolves into the dashboard's ambient background, so logging in feels like entering the app.
```

### Prompt 2.5: Pages: Verify Email, Forgot and Reset Password
```
Build:
1. /verify-email: shows "Check your inbox", has a resend button (60s cooldown), and handles ?token= to verify and then redirect to /login?verified=1.
2. /forgot-password: email field. ALWAYS show "If an account exists, we sent a link" (no user enumeration).
3. /reset-password?token=...: new password + confirm, with a strength meter; calls resetPassword(); invalid or expired token shows an error state with a link to request a new one.
All three use <AuthShell> with their own scene state:
- /verify-email: a 3D glass envelope floats and opens; a glowing letter rises out of it. On verification it folds into a check mark.
- /forgot-password: a 3D padlock with a key orbiting it.
- /reset-password: the key enters the lock and the shackle springs open when the password is saved.
```

---

## PHASE 3: App Shell and Dashboard

### Prompt 3.1: App Layout
```
Create app/(app)/layout.tsx:
- A collapsible sidebar: Dashboard, Upload Data, My Datasets, Projects, Reports, Chart Gallery, Settings.
- Top bar: global search (datasets/projects), theme toggle, and a user avatar dropdown (Profile, Settings, Logout).
- A mobile drawer version of the sidebar.
- Wrap everything in requireUser().

3D / PREMIUM UI:
- An ambient <GradientMeshBackground intensity={0.35}> behind the whole app (it slowly shifts; it must never distract from data).
- The sidebar is glass. The active item has a glowing pill indicator that slides between items (framer-motion layoutId). Icons do a small 3D flip on hover. Collapsing animates the width with a spring.
- A command palette (Ctrl/Cmd+K, cmdk library) that opens with a depth scale-in and backdrop blur. It contains: search datasets, jump to page, "Create chart…", "Upload file", toggle theme, and 3D quality.
- A notification bell with a 3D ring animation when a dataset finishes processing (also a toast with a mini <DataCrystal> success burst).
- <CursorSpotlight> active on all cards.
- Page transitions from Prompt 1B.4.
```

### Prompt 3.1b: Onboarding Tour
```
The first time a user logs in, run a guided tour (driver.js, restyled to glass) with a small <AIOrb> "guide" that floats next to each highlighted element. Steps: Upload → Data Quality → Charts → AI chat → Reports. It can be skipped and re-opened from the help menu.
```

### Prompt 3.2: Page: Dashboard (`/dashboard`)
```
Build app/(app)/dashboard/page.tsx (a server component):
- A welcome header with the user's name.
- Stat cards: Total datasets, Charts generated, Issues found/fixed, Reports shared.
- "Recent datasets" table: name, rows x cols, status badge (colour per status), issue count, created date, and actions (Open, Delete).
- A big "Upload new file" CTA card.
- An empty state for new users with a sample-dataset button ("Try with sample sales data") that loads a bundled demo .xlsx.
Functions in /lib/data/dashboard.ts: getDashboardStats(userId), getRecentDatasets(userId, limit=10), getActivitySeries(userId, days=30), getAvgQualityScore(userId).

3D DESIGN:
- A hero band with a <HoloBarChart3D> of the user's last-30-days activity (charts generated per day). Bars rise on load; hovering shows the date and count.
- Stat cards are <TiltCard>s with <FlipNumber> counters and a small animated glowing sparkline.
- An "Average data quality" <Gauge3D> that animates from 0 to the value.
- Recent datasets have a view toggle: "Table" (default, dense) or "Crystal shelf" (each dataset is a small <DataCrystal> on a glowing glass shelf, coloured by quality score; hovering floats it up with a name label; clicking moves it into the dataset page via a shared-element transition).
- The upload CTA card has a mini <PortalRing> that spins faster on hover.
- Empty state: a 3D floating spreadsheet with a pulsing "Drop your first file" prompt.
```

---

## PHASE 4: Upload

### Prompt 4.1: Page: Upload (`/upload`)
```
Build app/(app)/upload/page.tsx with a 3-step wizard:

STEP 1: Select files
- A drag-and-drop zone (react-dropzone) that accepts .xlsx, .xls, .csv, .tsv, .json, and .parquet, up to MAX_UPLOAD_MB each, max 5 files.
- Client checks: extension, size, empty file. Show a chip per file with its size and a remove button.
- A Project selector (existing project or "Create new").

STEP 2: Describe your data (optional but encouraged)
- Textarea: "What is this data about and what do you want to learn?" (sent to the AI as context).
- Optional selects: Industry (Sales, Finance, HR, Marketing, Operations, Healthcare, Education, Other), Goal (Compare, Trend, Composition, Distribution, Relationship, Flow, Geo, Let AI decide).

STEP 3: Upload and process
- Upload to S3 via a presigned URL with a per-file progress bar.
- Then call POST /api/datasets to create the Dataset record and queue the processing job.
- Redirect to /datasets/[id], which shows live processing status.

Functions:
- validateFileClientSide(file): {ok, error}
- getPresignedUploadUrl(fileName, mime, size)   -> API route /api/upload/presign (checks auth, type whitelist, size, and plan quota)
- uploadWithProgress(file, url, onProgress)
- createDatasetAction({storageKey, fileName, projectId, userContext, industry, goal})

3D DESIGN:
- The drop zone is a large <PortalRing>. At idle it swirls slowly. On drag-over it spins faster, glows brighter, and the particles get pulled toward the center. A wrong file type flashes red with a shake and a clear message.
- On drop, each file becomes a 3D glass sheet (with its icon and name) that flies in an arc INTO the portal. Upload progress is shown as a glowing ring filling around the portal, and each file chip keeps its own progress bar.
- The wizard steps change with a 3D card flip (rotateY) and a step indicator whose connecting line fills with light.
- In Step 2, the Goal options are small <TiltCard>s, each with an animated mini-chart icon for that goal.
- When all uploads finish, the portal flashes and the camera pushes through it into the processing view on /datasets/[id].
```

### Prompt 4.2: Processing Pipeline (Job Queue)
```
Create /web/lib/jobs/processDataset.ts (a BullMQ worker). The pipeline for one dataset:
1. status=PARSING      -> POST {ANALYSIS_SERVICE_URL}/parse
2. If the workbook has several sheets and none was chosen, pause with status=NEEDS_SHEET (add this status) so the UI can ask the user
3. status=VALIDATING   -> POST /validate  -> save ColumnProfile + ValidationIssue rows
4. status=ANALYZING    -> POST /recommend -> candidate charts with precomputed data
5. Call the AI layer (Phase 7) to rank, title, and explain the charts -> save ChartRecommendation rows
6. status=READY
On any failure: status=FAILED, store a user-friendly errorMessage, and log the stack trace.
Add GET /api/datasets/[id]/status for polling (every 2s), or Server-Sent Events.
Retry transient failures twice with exponential backoff.
```

---

## PHASE 5: Analysis Service (Python): Parsing and Profiling

### Prompt 5.1: `parse_file()`
```
In /analysis/app/services/parser.py implement:

parse_file(file_bytes, file_name, sheet=None) -> ParsedResult
- Detect the type by magic bytes, not only the extension.
- .xlsx/.xls: openpyxl/xlrd; list all sheets; read with data_only=True, but ALSO do a second pass with data_only=False to detect formula cells and Excel error values (#DIV/0!, #N/A, #REF!, #VALUE!, #NAME?, #NUM!, #NULL!).
- Detect merged cells and record their ranges.
- CSV/TSV: detect encoding (charset-normalizer) and delimiter (csv.Sniffer); handle BOM.
- JSON: support an array of records or {data:[...]}; flatten nested objects with json_normalize.
- detect_header_row(df_raw): the header may not be on row 1. Score the first 20 rows (most non-null string cells, next row types differ) and pick the best one. Report the rows skipped above it.
- Normalise headers: strip, collapse spaces, and dedupe ("Sales", "Sales_2"); keep originalName.
- Drop fully empty rows/columns but RECORD them as issues.
- Detect "total"/"subtotal"/"grand total" rows (a text cell matching /total/i and a numeric row ≈ sum of rows above) and flag them.
- Limits: max 1,000,000 rows. If larger, sample 200k rows for charts and say so.
Return: sheets[], selectedSheet, dataframe (stored as parquet in S3 for later steps), headerRowIndex, mergedRanges, formulaErrors[], parseWarnings[].

Endpoint: POST /parse {storageKey, sheet?} -> JSON summary.
Write pytest tests with fixture files: multi-sheet, header on row 4, merged cells, #DIV/0! cells, latin-1 CSV, semicolon CSV.
```

### Prompt 5.2: `profile_columns()` / `detect_column_type()`
```
In /analysis/app/services/profiler.py implement:

detect_column_type(series, name) -> (detectedType, semanticRole, confidence)
Rules, in order:
- BOOLEAN: ≤2 unique non-null values in {true/false, yes/no, y/n, 0/1}
- DATETIME: >80% of values parse with pd.to_datetime (try dayfirst both ways; flag ambiguity), or an Excel serial date (numbers 20000-60000 in a column named like date/time/day/month)
- CURRENCY: strings with $, €, £, ₹, ¥ or a name containing price/cost/revenue/amount -> convert to numeric
- PERCENT: strings ending in % or a name with pct/percent/rate and values 0-1 or 0-100
- NUMERIC/INTEGER: >90% castable to number after stripping thousands separators
- ID: unique ratio > 0.95 and name like id/code/sku/no/number, or sequential integers
- LAT/LON: name lat/latitude/lng/lon/longitude and values in range
- GEO_COUNTRY/GEO_STATE: >70% of values match a bundled ISO country list / US-state + Indian-state list
- CATEGORICAL: unique count ≤ 50 or unique ratio < 0.05
- TEXT: average length > 30 chars, or high cardinality strings
Semantic roles: detect OHLC (open/high/low/close), START_DATE/END_DATE (start/end/begin/finish/due), SOURCE/TARGET (from/to/source/target), and PARENT (parent/category/level).

profile_columns(df) -> list[ColumnProfile]
For each column: null count/%, unique count, top 10 values with counts, and sample values.
Numeric: min, max, mean, median, std, q1, q3, IQR, skewness, kurtosis, zero count, negative count.
Datetime: min, max, inferred frequency (daily/weekly/monthly/yearly), gaps.
Text: avg length, word count.

dataset_shape_summary(profiles) -> {numericCols, categoricalCols, datetimeCols, geoCols, textCols, hasOHLC, hasHierarchy, hasSourceTarget, hasStartEnd, rowCount}
```

---

## PHASE 6: Error Detection (the Validator)

### Prompt 6.1: `validate_dataset()`: every rule is its own function
```
In /analysis/app/services/validator.py create a rule-based validator. Each rule is a separate function with the signature
rule_xxx(df, profiles, context) -> list[Issue]
where Issue = {severity, code, column, rowIndexes[:100], affectedCount, message, suggestion, autoFixable}.
Row numbers shown to the user must be EXCEL row numbers (account for the header offset).

Implement ALL of these rules:

STRUCTURE
- rule_empty_file: no data rows -> ERROR
- rule_header_problems: missing/blank headers ("Unnamed: 3"), duplicate headers, numeric-looking headers -> WARNING
- rule_header_not_first_row: header found below row 1 -> INFO
- rule_merged_cells: merged ranges inside the data -> WARNING (values unmerged by forward-fill)
- rule_empty_rows_cols: fully blank rows/columns -> INFO, autoFixable
- rule_total_rows: embedded subtotal/total rows that would double-count -> WARNING, autoFixable
- rule_multiple_tables: large blank gaps suggesting 2+ tables in one sheet -> WARNING

CELL VALUES
- rule_excel_formula_errors: #DIV/0!, #N/A, #REF!, #VALUE!, #NAME?, #NUM! -> ERROR with exact cell addresses (e.g. "D14")
- rule_missing_values: null % per column; >50% ERROR, 5-50% WARNING, <5% INFO; suggest drop/impute (mean/median/mode/forward-fill for time series)
- rule_mixed_types: a numeric column containing text like "N/A", "-", "TBD", "twelve" -> ERROR, list the offending values
- rule_numbers_as_text: numbers stored as text / with thousands separators or currency symbols -> WARNING, autoFixable
- rule_whitespace: leading/trailing spaces, double spaces -> INFO, autoFixable
- rule_inconsistent_categories: the same category written differently ("USA", "usa", "U.S.A", "United States "). Use lowercase+strip grouping plus rapidfuzz similarity >90 -> WARNING, suggest a mapping
- rule_invalid_dates: unparseable dates, impossible dates (31/02), mixed formats (dd/mm vs mm/dd), future dates in "past" columns, dates before 1900 -> ERROR/WARNING
- rule_leading_zeros_lost: ID/phone/zip columns stored as numbers with lengths that vary -> WARNING
- rule_special_characters / encoding garbage (Ã©, �) -> WARNING

DUPLICATES
- rule_duplicate_rows: exact duplicate rows -> WARNING, autoFixable
- rule_duplicate_ids: an ID column with repeated values -> ERROR

STATISTICAL
- rule_outliers: IQR (1.5x) and |z|>3 on numeric columns -> WARNING, list the values and rows; do NOT auto-remove
- rule_negative_values: negatives in columns that should be positive (qty, price, age, count, revenue, units) -> WARNING
- rule_constant_column: one unique value -> INFO (useless for charts)
- rule_high_cardinality: a categorical column with too many unique values for a pie/bar -> INFO (suggest Top-N + "Other")

LOGICAL / CROSS-COLUMN
- rule_ohlc_consistency: high < low, open/close outside [low, high] -> ERROR
- rule_start_before_end: end date earlier than start date -> ERROR
- rule_percent_sum: percentage columns per row that should sum to 100 but don't (±1) -> WARNING
- rule_total_mismatch: a "Total" column ≠ sum of its component columns -> WARNING
- rule_geo_range: lat outside ±90, lon outside ±180, unknown country names -> ERROR/WARNING
- rule_time_gaps: a time series with missing periods or duplicate timestamps -> WARNING

validate_dataset(df, profiles, parse_meta) runs all rules, sorts by severity, and returns:
{ issues[], qualityScore (0-100, weighted: ERROR -10, WARNING -3, INFO -0.5, floor 0), summary: {errors, warnings, infos} }

Endpoint: POST /validate {datasetId}. Write one pytest per rule with a tiny fixture DataFrame.
```

### Prompt 6.2: `apply_fixes()`
```
In /analysis/app/services/cleaner.py implement apply_fixes(df, fixes[]) where each fix is {issueCode, column, action, params}.
Actions: trim_whitespace, convert_to_number, parse_dates(format), drop_duplicates, drop_empty, remove_total_rows, map_categories(mapping), fill_missing(strategy: mean|median|mode|ffill|value), drop_rows(rowIndexes), cap_outliers(method: iqr), rename_header.
- NEVER overwrite the original file. Save a new version (parquet) and keep a version history so the user can undo.
- After applying fixes, re-run profile + validate + recommend and return the new quality score.
Endpoint: POST /fix {datasetId, fixes[]}.
```

### Prompt 6.3: Page: Dataset Validation Report (`/datasets/[id]` → "Data Quality" tab)
```
Build app/(app)/datasets/[id]/page.tsx with tabs: Overview | Data Quality | Charts | Data Preview | AI Insights.

While status != READY: show a 3D PIPELINE VIEW with live polling (SSE preferred). A <DataCrystal> travels along a glowing track through 4 stations (Parse → Validate → Analyze → AI). The active station lights up with scanning particles, and live messages appear beside it ("Found header on row 4", "Detected 3 date columns", "14 issues found", "Scoring 212 chart candidates…"). When it reaches READY, the crystal bursts and the tabs slide in. On FAILED, the crystal cracks red and a clear error plus a "Retry" button appears. A plain stepper is the fallback at the LOW/OFF tier. If status=NEEDS_SHEET, show a sheet picker (sheet names + first 5 rows preview).

"Data Quality" tab:
- A <Gauge3D> quality score (0-100, red/amber/green) with <FlipNumber> counts of Errors / Warnings / Info.
- An <IssueTerrain> "error map": one row of instanced columns per dataset column × row buckets; spike height and colour show how many issues are in that part of the file. Hovering shows the column, the row range, and the issue count. Clicking a spike filters the issue list to that column and row range. When the user fixes issues, those spikes heal (red → green ripple) and the gauge animates up. That moment should feel rewarding.
- A filterable issue list (by severity, column, fixable). Each issue card shows: severity icon, message, column, affected count, the first 10 row numbers (clickable → jumps to that row in Data Preview with the cell highlighted), suggestion, and a "Fix" button when autoFixable (with a preview of the before/after diff in a dialog).
- A "Fix all safe issues" bulk button (only whitespace, numbers-as-text, empty rows, duplicates).
- For inconsistent categories: an editable mapping table (variant → canonical value).
- A "Download error report" button (Excel file with one sheet listing all issues + the original data with problem cells highlighted red/yellow).
- Version history dropdown with undo.

"Data Preview" tab: a virtualized table (TanStack Table + react-virtual), column type badges in the header, problem cells highlighted, sorting/filtering, and a column profile popover (stats + mini histogram).

"Overview" tab: file info, rows/cols, column list with detected type + role (the user can override a type via a dropdown → re-run recommend), and a sheet switcher.
```

---

## PHASE 7: Chart Recommendation Engine

### Prompt 7.1: Chart Catalogue (single source of truth)
```
Create /analysis/app/services/chart_catalog.py AND a mirrored /web/lib/charts/catalog.ts containing all 45 chart types.
Each entry: {id, name, family, description, requires, optional, minRows, maxCategories, bestFor, avoidWhen, echartsSeriesType}.

Use these data requirements (C = categorical, N = numeric, T = datetime, G = geo, TXT = text):

COMPARISON
- bar: 1C + 1N (aggregated); horizontal when labels are long or categories > 8
- grouped_bar: 2C + 1N, or 1C + 2-5N; second C ≤ 6 values
- stacked_bar: 2C + 1N (parts add up meaningfully)
- stacked_bar_100: 2C + 1N; compare shares, not totals
- radial_bar: 1C (≤ 12) + 1N
- lollipop: 1C (many, 10-50) + 1N, ranked

COMPOSITION
- pie: 1C (≤ 6) + 1N, all values positive
- donut: same as pie, with a KPI in the center
- treemap: 1-3 hierarchical C + 1N positive
- sunburst: 2-4 hierarchical C + 1N positive
- stacked_area: 1T + 1C (≤ 8) + 1N
- waffle: 1C (≤ 5) + 1N, percentages

DISTRIBUTION
- histogram: 1N (≥ 20 rows); bins via Freedman-Diaconis
- box_plot: 1N, optionally split by 1C (≤ 15)
- violin: 1N + 1C (≤ 8), ≥ 30 rows per group
- density: 1N (≥ 30 rows), KDE computed in Python (scipy gaussian_kde)
- dot_plot: 1N (< 100 rows) or 1C + 1N
- ridge: 1N + 1C (3-15 groups), a KDE per group

RELATIONSHIP
- scatter: 2N (optional C for colour)
- bubble: 3N (x, y, size) + optional C
- heatmap: 2C + 1N, or T(day x hour) + N
- hexbin: 2N with > 1000 rows (bin in Python)
- correlogram: ≥ 3N; Pearson + Spearman matrices

TIME SERIES
- line: 1T + 1-5N, or 1T + 1C (≤ 8) + 1N
- area: 1T + 1N
- candlestick: T + OHLC roles (+ optional volume)
- sparkline: 1T + many N (one mini line per metric, KPI grid)
- streamgraph: 1T + 1C (3-15) + 1N

FLOW / HIERARCHY / NETWORK
- sankey: SOURCE + TARGET + N (or 2-4 C stages + N)
- chord: 2C forming a square matrix (from/to) + N
- network: SOURCE + TARGET (+ optional weight); ≤ 500 nodes
- dendrogram: PARENT/child columns, or ≥ 3N for hierarchical clustering (scipy linkage)
- funnel: 1C ordered stages (3-8) + 1N decreasing

SPECIALIZED
- radar: 1C (entities ≤ 6) + 3-10N (normalised 0-1)
- waterfall: 1C (ordered steps) + 1N with +/- values
- gantt: TASK (C) + START_DATE + END_DATE (+ optional C for owner/status)
- word_cloud: 1 TXT column (tokenise, remove stopwords, top 150 words)
- bullet: 1C + actual N + target N (+ optional ranges)
- choropleth: GEO_COUNTRY or GEO_STATE + 1N
```

### Prompt 7.2: `recommend_charts()`
```
In /analysis/app/services/recommender.py implement:

get_eligible_charts(shape_summary, profiles) -> list[Candidate]
- Match every catalogue entry's `requires` against the column profiles. Generate ALL valid column combinations, capped at 5 per chart type (prefer columns with fewer nulls and higher variance; prefer METRIC roles over IDs; never use ID columns as metrics).

score_candidate(candidate, profiles, user_goal) -> 0-100
- Base rules, highest priority first:
  1. Datetime + 1 metric → line (base 95)
  2. 1 categorical + 1 metric → bar (base 90)
  3. 2 numeric → scatter (base 85)
  4. 1 numeric → histogram (85) / box plot (80)
  5. OHLC present → candlestick (98)
  6. Geo column → choropleth (92)
  7. Source+Target → sankey (92)
  8. Start+End dates → gantt (95)
- Penalties: pie with > 6 slices (-40), bar with > 30 categories (-30, suggest Top-N), negative values in part-to-whole charts (disqualify), too few rows for distribution charts (disqualify), > 40% missing in a used column (-25).
- Bonus: +15 if the chart family matches the user's goal from upload Step 2.

build_chart_data(candidate, df) -> chartData
One transformer function per chart type in /analysis/app/services/transformers/<family>.py:
aggregate_bar, aggregate_grouped, aggregate_stacked, normalize_100, top_n_with_other, compute_histogram_bins, compute_box_stats, compute_kde, compute_violin, compute_ridge, compute_hexbin, compute_correlation_matrix, resample_timeseries (auto-choose day/week/month by range), build_ohlc, build_hierarchy_tree, build_sankey_links, build_chord_matrix, build_network_graph, compute_dendrogram, build_funnel, normalize_radar, build_waterfall_steps, build_gantt_tasks, extract_word_frequencies, build_bullet, build_choropleth.
Output must be compact JSON ready for ECharts (downsample to ≤ 2,000 points using LTTB for lines and random sampling for scatter).

recommend_charts(datasetId) -> top 40 candidates sorted by score, each with {chartType, family, columns mapping, chartData, score, ruleReason}.
Endpoint: POST /recommend. Write tests: sales data → bar/line top; stock data → candlestick top; survey data → histogram/box.
```

---

## PHASE 8: AI Layer (Claude)

### Prompt 8.1: `aiRankAndExplain()`
```
Create /web/lib/ai/claude.ts using @anthropic-ai/sdk (model "claude-sonnet-5", server only).

Function aiRankAndExplain({userContext, industry, goal, profiles, validationSummary, candidates, sampleRows}) -> AiResult
- Build the prompt with: the dataset description, the column profiles (name, type, role, key stats), the top 10 validation issues, the candidate list (id, chartType, columns, ruleReason, score) WITHOUT the chartData, and ≤ 30 sample rows.
- Force structured JSON output (use tool use / a JSON schema) with this shape:
  {
    datasetSummary: string (3-5 sentences, plain language),
    selectedCharts: [{candidateId, rank, title, reason (why this chart suits this data, 1-2 sentences), insight (the key takeaway, which MUST quote only numbers present in the provided stats)}],   // pick 8-12, spread across families
    dataQualityNarrative: string (explain the errors found and their impact on the charts),
    suggestedQuestions: string[5]  (follow-up questions the user could ask)
  }
- Validate the response with zod. On a parse failure, retry once; if it still fails, fall back to rule-based ranking with templated titles.
- System prompt: "You are a data visualization expert. Choose charts only from the candidates provided. Never invent numbers or columns. Prefer clarity over novelty. Avoid pie charts with many slices. Mention data-quality caveats when a chart uses a column with issues."
- Log token usage per user and enforce a monthly AI quota by plan.

Function aiGenerateInsight(chartData, chartType, title) -> string
- Called on demand from the "Explain this chart" button; gets a summarised version of chartData (aggregates only).

Function aiAskData(datasetId, question, history) -> {answer, chartSuggestion?}
- Chat about the dataset. The AI can request a chart via a tool `create_chart({chartType, columns, filters})`; the server validates it against the catalogue, calls the Python /chart endpoint to build the data, and returns a rendered chart in the chat.
```

---

## PHASE 9: Charts UI

### Prompt 9.1: `<ChartRenderer />` and Per-Type Option Builders
```
Create /web/components/charts/ChartRenderer.tsx: props {chartType, chartData, config, title, height}.
It looks up an option builder from /web/lib/charts/builders/<chartType>.ts — one pure function per chart, e.g.:
buildBarOption, buildGroupedBarOption, buildStackedBarOption, buildStacked100Option, buildRadialBarOption, buildLollipopOption,
buildPieOption, buildDonutOption, buildTreemapOption, buildSunburstOption, buildStackedAreaOption, buildWaffleOption (custom series / grid of squares),
buildHistogramOption, buildBoxPlotOption, buildViolinOption (custom series from KDE), buildDensityOption, buildDotPlotOption, buildRidgeOption,
buildScatterOption, buildBubbleOption, buildHeatmapOption, buildHexbinOption (custom polygons), buildCorrelogramOption,
buildLineOption, buildAreaOption, buildCandlestickOption (+volume sub-grid, dataZoom), buildSparklineGrid, buildStreamgraphOption (themeRiver),
buildSankeyOption, buildChordChart (d3-chord component), buildNetworkOption (force layout), buildDendrogramOption (tree series), buildFunnelOption,
buildRadarOption, buildWaterfallOption (stacked transparent-base bars), buildGanttOption (custom series with time axis), buildWordCloudOption, buildBulletOption, buildChoroplethOption (register world / country GeoJSON).

Common features for every chart: responsive resize, theme-aware colours (a colour-blind-safe palette), tooltips with formatted numbers (K/M/B, currency, %), a legend, empty-state handling, and "insufficient data" messages.
Toolbar per chart: switch to a compatible chart type (dropdown of other eligible types for the same columns), change column mapping, Top-N slider, sort, Export (PNG, SVG, CSV of the chart data), Fullscreen, Pin to report, "Explain this chart" (AI).
Write a Storybook or /dev/charts page showing all 45 charts rendered with mock data so each one can be checked visually.

3D CHART MODE (echarts-gl + three.js):
- Every chart card has a "2D / 3D" toggle for types that have a meaningful 3D form:
  bar/grouped/stacked → bar3D (category × series × value); scatter/bubble → scatter3D (with a third numeric column or a colour group); heatmap/hexbin → a bar3D height grid or a surface; line/area → line3D ribbons in depth per series; correlogram → a 3D matrix of bars; pie/donut → a 3D extruded pie (custom three.js; the hovered slice lifts out); choropleth → a globe (echarts-gl globe or <DataGlobe>) with extruded country bars; network → <NetworkConstellation>; sunburst/treemap → 3D stacked-layer treemap.
- Types without a meaningful 3D form (e.g. box plot, gantt, bullet) stay 2D but get depth styling: a glass background, glowing strokes, and soft shadows.
- The 3D charts support orbit (drag), zoom, auto-rotate (off with reduced motion), and a "reset view" button. Hovering highlights and shows a tooltip.
- Entrance animations for ALL charts: bars grow from the axis with a stagger, lines draw along their path, pie slices sweep in, and scatter points pop in with a scale spring. They run once when the chart scrolls into view.
- IMPORTANT: 3D is never the only way to read a chart. 2D stays the default for accuracy, and a "View data" button shows the underlying table.
```

### Prompt 9.1b: Immersive View and Chart Galaxy
```
- "Immersive View" (fullscreen from any chart): the chart floats in a dark 3D room with a reflective floor (drei MeshReflectorMaterial on HIGH, a plain floor otherwise). The camera orbits in, and the AI insight appears on a floating glass panel beside the chart. Arrow keys move to the next or previous chart with a smooth camera fly-through.
- "Chart Galaxy" view (a toggle on the Charts tab): all recommended charts float as glass panels arranged by family in clusters, like a constellation. The user can fly around (orbit + zoom) and click a panel to focus it (the camera flies to it and the panel becomes interactive). Panels far from the camera render as cached textures (render each ECharts instance to a canvas texture) so it stays at 60 fps with 40 charts.
- Both views need keyboard navigation and Esc to exit, and both fall back to a regular fullscreen chart at the LOW/OFF tier.
```

### Prompt 9.2: Page: Charts Tab (`/datasets/[id]` → "Charts")
```
Build the Charts tab:
- An AI summary card at the top (datasetSummary + data-quality caveat banner if the quality score < 70).
- Filter chips by family: All, Comparison, Composition, Distribution, Relationship, Time-Series, Flow & Hierarchy, Specialized.
- A responsive grid (1/2/3 cols) of ChartCards: title, family badge, score, the chart, the AI "why this chart" reason (collapsible), and the insight.
- A drag-and-drop reorder (dnd-kit) that persists `order`.
- An "Add chart" button → modal: pick any of the 45 chart types (greyed out with a tooltip explaining the missing requirement if the data is not eligible), pick columns, preview, save.
- A "Build report" button → select pinned charts → create a Report.

3D DESIGN:
- ChartCards are <TiltCard>s (a max tilt of 4° so charts stay readable) with a cursor spotlight and a gradient border that glows by family colour.
- Changing the family filter animates the grid (framer-motion layout): cards fly to their new positions and filtered-out cards shrink back in depth.
- Dragging a card lifts it with a stronger shadow and scale 1.03; the other cards make room smoothly.
- Switching chart type (toolbar dropdown) morphs the chart (ECharts universalTransition: bars morph into pie slices, and so on).
- A view switcher at the top: Grid | Chart Galaxy (Prompt 9.1b) | Dashboard (Prompt 10B.5).
- In the "Add chart" modal, all 45 chart types show as small <TiltCard>s with animated icons; hovering an ineligible one shows why it can't be used ("needs a date column").
```

### Prompt 9.3: Page: AI Insights Tab and Chat
```
Build the "AI Insights" tab:
- The dataset summary, a data-quality narrative, and key insights across the charts as a bullet list.
- A chat panel (streaming responses) using aiAskData. Show suggestedQuestions as clickable chips. When the AI returns a chart, render it inline with a "Save to charts" button.
- Persist the chat history per dataset.

3D DESIGN:
- An <AIOrb> sits at the top of the chat panel: it breathes at idle, swirls while thinking, and ripples with the streaming token rate while answering.
- Insight cards reveal one by one (stagger + blur-in). Each has a coloured glow for its type (trend ↑ green, risk ↓ red, anomaly amber) and a "Show me" button that scrolls to and pulses the related chart.
- Charts the AI creates in the chat assemble in place (particles converge into the chart, then the real ECharts chart fades in).
- Voice input button (Web Speech API, where the browser supports it): speaking makes the orb react to the mic level.
```

---

## PHASE 10: Remaining Pages

### Prompt 10.1: My Datasets (`/datasets`) and Projects (`/projects`, `/projects/[id]`)
```
- /datasets: a searchable, sortable, paginated table of all the user's datasets (name, project, rows, quality score, status, date) with bulk delete and re-analyze.
- /projects: project cards (name, dataset count, last updated); create/rename/delete.
- /projects/[id]: the project's datasets; "Compare datasets" (when 2+ share columns, build combined charts, e.g. a line per file); upload into the project.
Deleting a dataset removes its S3 objects + DB rows (with a confirmation dialog that asks the user to type the name).

3D DESIGN:
- /datasets: Table view (default) plus a "Crystal grid" view (a <DataCrystal> per dataset, coloured by quality score, sized by row count). Deleting plays a shatter animation.
- /projects: project cards are <TiltCard>s with a stacked-sheets 3D depth effect showing how many datasets each one holds.
- /projects/[id]: a "Data Map" view using <NetworkConstellation>: datasets are nodes and links are shared columns (thicker = more overlap). Clicking two nodes starts "Compare datasets".
```

### Prompt 10.2: Reports (`/reports`, `/reports/[id]`, public `/r/[shareToken]`)
```
- /reports: a list of reports.
- /reports/[id]: a report editor: title, reorderable charts, a text block between charts (markdown), the AI executive summary (generated with a button), and Export to PDF (server-side Playwright render) and PPTX (pptxgenjs, one chart image per slide).
- Share: toggle public → generate shareToken → a read-only public page /r/[token] (no login, no raw data download, charts only).

3D DESIGN:
- "Present" mode: a fullscreen slideshow with 3D transitions between slides (depth push, cube rotate, or fly-through; the user picks one). Presenter notes, a laser-pointer cursor, and keyboard/clicker support.
- The public /r/[token] page opens with a cinematic cover (the report title over a slow <ParticleMorph> that forms the first chart), then scrolls through the charts with scroll-reveal animations.
- Exports: "Animated export" records a 3D chart rotation or the slide transitions to MP4/WebM (MediaRecorder on the canvas stream) or GIF, alongside the PDF and PPTX.
- Branding: upload a logo and pick brand colours; they are applied to charts, slides, PDFs, and the public page.
```

### Prompt 10.3: Chart Gallery (`/gallery`)
```
A reference page listing all 45 charts grouped by the 7 families: a thumbnail (rendered with mock data), when to use it, the data it needs (e.g. "1 date column + 1 number column"), and a "Download sample Excel template" button that generates an .xlsx with the right column structure for that chart (use exceljs).
3D DESIGN: the top of the page is a <Carousel3D> ring of the 7 families; spinning it to a family filters the grid below with a layout animation. Every chart card is a <TiltCard> with a live animated mini-chart (rendered only while in view). A "Compare" drawer shows two charts side by side with the same data so users can see when to use which.
```

### Prompt 10.4: Settings (`/settings`)
```
Tabs:
- Profile: name, avatar upload, email (change requires re-verification).
- Security: change password (current password required), connected Google account, active sessions with "log out all", delete account (type DELETE to confirm; removes all data).
- Preferences: default theme, number format (1,234.56 vs 1.234,56), date format (DD/MM/YYYY vs MM/DD/YYYY; used as the parser hint), currency, and colour palette.
- Usage & Plan: uploads this month, AI tokens used, storage used, and plan limits (FREE: 10 uploads/month, 10MB; PRO: unlimited, 50MB). Show the usage as three <Gauge3D> rings.
- Appearance (NEW, important): a "3D effects" setting (Auto (recommended) / High / Medium / Low / Off), a "Reduce motion" switch, an ambient background intensity slider, and accent colour presets. Show a live preview panel with a small 3D scene that updates instantly. Save the choices to the user's profile AND to localStorage so they apply before the page hydrates (no flash).
- 3D details: the profile avatar sits in a glowing rotating ring; the theme toggle is a 3D sun that morphs into a moon; the Delete Account dialog shows a <DataCrystal> cracking as the user types DELETE.
```

### Prompt 10.5: Admin (`/admin`), optional
```
ADMIN role only: users list, usage stats, failed jobs with error logs and a retry button, and AI cost per day.
```

---

## PHASE 10B: Premium Features (what makes it more than a regular app)

### Prompt 10B.1: Natural-Language Chart Builder
```
Add a "Describe a chart" input (in the command palette and on the Charts tab), e.g. "monthly revenue by region as a stacked area, 2024 only".
Function aiParseChartRequest(text, profiles) -> {chartType, columns, filters, aggregation, timeGrain}. It uses Claude with a strict JSON schema, and the result is validated against the catalogue and the real column names (fuzzy-match column names; ask a clarifying question if it's ambiguous). Then call the Python /chart endpoint and render the chart with the particle "assemble" animation.
```

### Prompt 10B.2: Forecasting and Anomaly Detection
```
Python /analysis/app/services/forecast.py:
- forecast_series(df, timeCol, metricCol, horizon) -> uses statsmodels ExponentialSmoothing / SARIMAX (auto-select by AIC); returns the forecast + 80%/95% confidence bands + a backtest MAPE.
- detect_anomalies(series) -> rolling z-score + IsolationForest; returns the anomalous points with a severity score.
UI: a "Forecast" toggle on line/area charts draws the forecast as a dashed line with a glowing confidence band that grows in; anomalies show as pulsing glow markers with a tooltip, and the AI explains the biggest anomaly. Always show the accuracy (MAPE) so users know how much to trust it.
```

### Prompt 10B.3: What-If Simulator
```
A "What-if" panel on the dataset page: the user picks metric columns and moves sliders (e.g. "Price +10%", "Units −5%", or a custom formula). Every chart recalculates live (debounced; aggregates recomputed in a web worker from the cached dataset). Changed values are highlighted and a delta badge appears on each chart. Scenarios can be saved and compared side by side.
```

### Prompt 10B.4: AI Data Story (Scrollytelling)
```
A "Generate story" button: Claude writes a 5-8 chapter narrative that uses ONLY the computed stats and the chosen charts. Render it as a scroll-driven story page: each chapter pins a chart while the text scrolls, the camera transitions between charts in 3D, and key numbers count up. It can be shared publicly (like reports) and exported to video.
```

### Prompt 10B.5: Interactive Dashboard Builder with Cross-Filtering
```
A "Dashboard" view: a drag-and-resize grid (react-grid-layout) of the user's charts, KPI tiles (<FlipNumber>), and text blocks. Cross-filtering: clicking a bar, slice, or point filters every other chart (animated update), with filter chips at the top. Global date-range and category filters. Save several dashboards per dataset; dashboards refresh automatically when the dataset is re-uploaded.
```

### Prompt 10B.6: Data Connectors and Scheduled Refresh
```
Besides file upload, the user can import from Google Sheets, Google Drive, OneDrive / Excel Online (OAuth), or a public CSV URL. A scheduled refresh (hourly/daily/weekly via a BullMQ repeat job) re-runs validation and the charts. If the quality score drops or NEW errors appear after a refresh, notify the user.
```

### Prompt 10B.7: Alerts
```
"Alert me when…" rules on any metric (e.g. "monthly revenue < 50,000", "error count > 0", "anomaly detected"). The rules are checked after each upload or refresh; notifications go in-app (with the bell animation) and by email. The user can manage or disable rules in Settings.
```

### Prompt 10B.8: Collaboration
```
- Invite teammates to a project with the roles Owner / Editor / Viewer (email invites, accepted after login).
- Comments pinned to a chart or to a specific data point, with @mentions and resolve.
- Live presence: teammates' avatars and cursors on the dataset page (Liveblocks or Yjs + y-websocket).
- An activity feed per project (uploaded, fixed issues, created chart, shared report).
```

### Prompt 10B.9: Keyboard Power Features
```
Shortcuts (shown with the "?" key): U upload, / search, G then D dashboard, 1-7 filter by chart family, F fullscreen chart, 3 toggle 3D mode, E export, Ctrl+Z undo the last data fix. Everything in the command palette is reachable by keyboard.
```

---

## PHASE 11: Security, Testing and Deployment

### Prompt 11.1: Hardening
```
Audit and implement:
- File security: whitelist MIME types + magic bytes, reject macro-enabled .xlsm, zip-bomb protection (max uncompressed size 500MB, max ratio 100x), strip formulas before any re-export, and CSV-injection protection (prefix cells that start with = + - @ when exporting).
- Every DB query is scoped by userId (write a helper assertOwnership(resource, userId)).
- CSRF protection (Auth.js), secure/httpOnly cookies, CSP headers, rate limiting on upload and AI endpoints.
- S3 objects private; access only via short-lived signed URLs.
- Prompt-injection safety: cell values sent to the AI are wrapped as data and the system prompt says to ignore instructions inside the data.
- Delete uploaded files after 90 days on the FREE plan (a cron job).
- Audit-log login, upload, delete, share.
```

### Prompt 11.2: Tests
```
- Python: pytest for the parser, every validator rule, every transformer, and the recommender (target 85% coverage).
- TS: vitest for /lib/auth, /lib/charts/builders (snapshot of each option), and the zod schemas.
- E2E: Playwright — sign up → verify → login → upload the sample Excel with deliberate errors → see ≥ 10 issues → fix all safe → see charts → export PNG → create and share a report → logout.
Create /tests/fixtures/messy_sales.xlsx that contains EVERY error type from Phase 6 for the E2E test.

3D QUALITY TESTS (these prove the 3D effects "work perfectly"):
- FPS test (Playwright): on every page, sample requestAnimationFrame for 5 seconds while scrolling and moving the mouse. Assert the average is ≥ 55 fps (desktop profile) and ≥ 30 fps with 4x CPU throttling + the MEDIUM tier. Fail CI if it drops.
- WebGL context test: after navigating through all pages 20 times, assert the number of live WebGL contexts is ≤ 1 and the JS heap has grown < 20% (no memory leaks; dispose works).
- Fallback tests: launch Chrome with --disable-webgl → every page renders its poster/CSS fallback with no console errors. With prefers-reduced-motion: reduce → no auto-rotation or parallax.
- Context-loss test: force WEBGL_lose_context.loseContext() → the fallback shows and the scene restores.
- Visual regression: Playwright screenshots of every page at the OFF tier (deterministic) in both themes, compared against a baseline.
- Lighthouse CI budgets: Performance ≥ 85, Accessibility ≥ 95, LCP < 2.5s, CLS < 0.05, TBT < 200ms on the landing, login, dashboard, and dataset pages.
- Real-device check: a test matrix doc (iPhone Safari, Android Chrome, Windows Chrome/Edge with integrated GPU, Mac Safari) with a checklist for each 3D component.
```

### Prompt 11.3: Deploy
```
Production setup:
- web → Vercel (or Docker on a VPS); analysis service → Docker on Railway/Render/Fly.io; Postgres → Neon/Supabase; Redis → Upstash; storage → AWS S3 / Cloudflare R2.
- GitHub Actions CI: lint, typecheck, tests, build, and Prisma migrate on deploy.
- Sentry for errors in both services, and structured logging.
- Write DEPLOY.md with step-by-step instructions.
```

---

## Quick Reference: Build Order Checklist

| # | Phase | Output |
|---|-------|--------|
| 0 | Master context | AI understands the project |
| 1 | Setup + DB | Running skeleton |
| 1B | 3D design system | Theme, 3D engine, 17 reusable 3D/motion components, /dev/3d playground |
| 2 | Auth pages | Signup / Login / Verify / Reset |
| 3 | Shell + Dashboard | Logged-in home |
| 4 | Upload + Queue | Files stored, job running |
| 5 | Parse + Profile | Column types detected |
| 6 | Validator + Fixes + Quality page | **Errors found and fixable** |
| 7 | Catalogue + Recommender | 45 charts mapped to the schema |
| 8 | Claude AI layer | Ranked charts, explanations, chat |
| 9 | Chart renderer + Charts tab | Charts on screen |
| 10 | Datasets, Projects, Reports, Gallery, Settings | Complete app |
| 10B | Premium features | NL charts, forecasting, what-if, data stories, dashboards, connectors, alerts, collaboration |
| 11 | Security, Tests, Deploy | Production |

**Tip:** When something breaks, don't paste the next prompt yet. Send the AI this: *"Here is the error: [paste]. Fix only this, keep all other code unchanged, and explain the cause."*

**3D tip:** If a 3D effect stutters, flickers, or goes black, send the AI this: *"The [component] on [page] is [dropping frames / flickering / blank]. Follow the 3D Performance Rules in the Master Context. Check for extra Canvas instances, missing dispose, per-frame React state updates, non-instanced meshes, and postprocessing on low tiers. Fix it, then confirm it holds 60 fps in /dev/3d."*
