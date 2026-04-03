# SecondTake.ai

> Rescue flopped videos with data-backed redesign. Paste a YouTube link → pull stats + transcript → Snowball research finds what actually wins in that niche → AI scores outlier relevance → vision analysis extracts thumbnail patterns → generates new packaging directions with titles and thumbnail concepts → side-by-side comparison with CTR prediction.

**Live app:** [secondtakeai.netlify.app](https://secondtakeai.netlify.app)

---

## The Problem

Video packaging — title, thumbnail, hooks — is responsible for 80% of view count variance. A video that flopped isn't always bad content. It's usually bad packaging.

Most creators guess what to change. SecondTake doesn't guess. It runs multi-round research across YouTube, finds statistical outliers in your niche, extracts what's working visually and structurally, then generates new packaging calibrated to how badly your video underperformed.

This is the third evolution of the tool. V1 was a monolithic HTML file. V2 added thumbnail generation. V3 (this) is a full SvelteKit app with server-side key management, type-safe state, a 5-screen user journey, and a canvas editor that lets you compose thumbnails from decomposed outlier elements.

---

## How It Works — 5 Screens

### 1. Input — Paste a YouTube URL

![Home](./docs/packaging-reviver/01-home-paste-url.png)

Paste a link and hit Revive Packaging. The app pulls video metadata from the YouTube Data API — title, thumbnail, channel stats, publish date. Your saved projects appear at the bottom so you can pick up where you left off.

![Stats](./docs/packaging-reviver/02-video-analysis-stats.png)

Performance stats load automatically. Views, subscriber count, video age, and the channel average multiplier — 0.34x here means this video got 34% of the channel's average. That multiplier feeds the Change Dial later.

![Topics](./docs/packaging-reviver/03-transcript-key-topics.png)

Paste the transcript and Gemini extracts up to 20 key topics — specific subjects, people, activities, and themes discussed in the video. These topics feed directly into seed query generation, making the Snowball research phase dramatically more targeted than title-only analysis. Transcript is capped at 5,000 characters with intro content prioritized.

---

### 2. Research — Snowball Engine

The research system runs multi-round iterative YouTube searches. Each round generates queries, searches YouTube, detects statistical outliers, and feeds winners into the next round.

![Queries](./docs/packaging-reviver/05-seed-queries-with-controls.png)

**Seed Queries**: Gemini generates 30 seed queries by analyzing the title, description, and key topics from the transcript. Each query has expand/narrow/similar controls. The green "Script 20" badge confirms transcript data is being used. Remove irrelevant queries, expand promising angles, or add your own before running Round 1.

![Round 1](./docs/packaging-reviver/05-round1-outlier-results.png)

**Round 1 Results**: All seed queries run against YouTube. Results show outlier videos with multiplier badges (7.5x, 25x, 20x — how many times above their channel average each video performed). Filter by No Shorts, No Clips. Each query tab shows its result count.

**Outlier Detection** uses 3 layers of filtering:
- **Hard reject** — Duration limits, category exclusions, language filtering
- **Soft flags** — Subscriber floor (views/subs must be at least 5%), re-upload detection, unlisted/premiere warnings
- **AI relevance scoring** — Gemini vision analysis of each thumbnail + strategic-fit evaluation. Every outlier gets scored on topical relevance (would viewers of your video click this?) and thumbnail composition relevance (can this visual style inspire a better thumbnail?)

![Round 2 Queries](./docs/packaging-reviver/06-round2-refined-queries.png)

**Round 2 — Snowball Effect**: Round 1 winners generate Round 2 queries. Gray queries are from Round 1, green are new refined queries snowballed from the winners. This is where research narrows from broad to surgical.

![Round 2 Results](./docs/packaging-reviver/07-round2-results-relevance.png)

**Round 2 Results** with full relevance analysis. Each card shows: thumbnail, multiplier, channel name, view count, and Gemini's reasoning — why it was selected or rejected, topical match, and visual composition notes. 327 outliers found across both rounds, 20 selected for packaging generation.

---

### 3. Strategy — Diagnosis + Directions

![Diagnosis](./docs/packaging-reviver/08-diagnosis-strategy.png)

**The Change Dial** calibrates edit intensity based on the underperformance ratio:
- Ratio > 1.0 → MINIMAL — 95%+ stays unchanged
- 0.8–1.0 → LOW — 90%+ stays unchanged
- 0.5–0.8 → MEDIUM — 50-70% preserved
- Below 0.5 → HIGH — Channel identity preserved, everything else is negotiable

At 0.34x, this video gets HIGH — major surgery needed.

**Diagnosis** outputs three tag categories, all editable:
- **Preserving tags** (green) — What's already working. MrBeast face, school imagery, global flags, country names.
- **Fixing tags** (red) — What needs to change. Generic school visuals, vague title, missing emotional hook, lack of specific impact.
- **Recommended edits** (blue) — Specific actions. Show the "before" state in thumbnail, add child labor visual for emotional hook, emphasize "clean water," reduce MrBeast to 30% of frame, add specific impact number to title.

Each tag includes supporting evidence from the outlier research — not opinions, data.

![Title Analysis](./docs/packaging-reviver/10-title-analysis-packaging-directions.png)

**Title Pattern Analysis** detects formulas from the outliers (e.g., "I [action] for [duration] in [location]"), counts how many outliers use each pattern, and generates new title variations based on what's statistically proven to work.

**Packaging Directions** are complete strategy proposals. Each direction includes a new title, a thumbnail concept description, a format archetype (e.g., "close-up emotional reaction," "information abundance"), layout composition principles, a stakes ladder showing emotional progression, and proof videos showing the pattern works.

![Directions](./docs/packaging-reviver/11-recommended-directions-with-thumbnails.png)

Each direction comes with reference thumbnails sourced exclusively from the outlier research — not arbitrary uploads. These are the videos that proved the pattern works at scale.

---

### 4. Generate — Canvas Editor + AI Thumbnails

![Generate](./docs/packaging-reviver/12-direction-tab-generate.png)

Each direction gets its own tab. Select references, hit Generate, and Gemini creates a new thumbnail informed by the edit directions, the Change Dial intensity, and the outlier visual patterns.

**Multi-Model Support**:
- **NanoBanana Pro** (gemini-3-pro-image-preview) — Primary model
- **NanoBanana 2** (gemini-3.1-flash-image-preview) — Fallback
- **NanoBanana OG** (gemini-2.5-flash-image-preview) — Legacy

**Generation Controls**: Model selector, aspect ratio (16:9, 1:1, 4:3, 9:16), quality tiers (standard/high/ultra), batch generation (1–5 variations), and parent selector to branch from any previous iteration.

**Canvas Editor** with 5 element types:
- **Background** — Full canvas fill (solid, gradient, image)
- **Subject** — Main character/face layer with face swap support
- **Object** — Props, graphics, UI elements
- **Text** — Titles, captions with font size, weight, color, shadow, outline controls
- **Reading Cue** — Visual attention guides (arrows, circles, highlights)

Full layer management: drag/resize, z-order control, visibility toggle, lock/unlock, per-layer opacity, rotation, and filters (brightness, contrast, saturation, hue, blur). 50-state undo/redo history.

**Layer Decomposition**: Break any reference thumbnail into component parts — subject, background, text, objects — each with bounding boxes and metadata. Auto-compose arranges extracted layers intelligently on canvas.

**Relight Modal**: 3D sphere UI with 9 light directions, 7 color options, brightness/intensity/mood controls for relighting generated thumbnails.

**Face Swap**: Upload 1–4 creator portraits. The app uses Replicate's face-swap model to inject your face into generated thumbnails while preserving the composition. Pre-swap backup stored for undo.

![Generated](./docs/packaging-reviver/13-generated-thumbnail-with-gallery.png)

Generated thumbnail appears alongside the original. Every iteration is saved with full lineage — direction ID, parent iteration, model used, prompt, canvas snapshot, and identity score. You can revert, compare, or branch from any version.

**Resolution Export**: 1K, 2K, or 4K upscale for any generated thumbnail.

---

### Wild Card Mode

![Wild Card](./docs/packaging-reviver/14-wild-card-mode.png)

Wild Card sends ALL previous generations + outlier data to the AI and asks for something nobody's tried. It assembles context from scanned outliers, loaded directions, diagnosis tags, script topics, and every previous generation — then produces 4 completely new concepts in a single batch.

![Wild Card Generated](./docs/packaging-reviver/15-wild-card-generated.png)

Results show Current vs Remix side-by-side with the latest generation in the gallery. Each wild card gets a unique concept name. Previous attempts are tracked so each batch builds on what came before — no repeated ideas.

Wild cards are tagged separately and filterable in the Compare screen.

---

### 5. Compare + Export

![Compare](./docs/packaging-reviver/16-compare-original-vs-new.png)

Side-by-side comparison of original and new packaging. Filter by Direction, Wild Card, Imported images, or Selected only.

**Vision Model Scoring** — Each generated thumbnail is scored on 8 dimensions:
1. Market size
2. Difficulty
3. Negative angle
4. Readability
5. Emotion
6. Contrast
7. Curiosity gap
8. Composition

CTR improvement prediction based on the aggregate score.

![Export](./docs/packaging-reviver/17-compare-select-export.png)

**Export Formats**:
- **PNG/JPG** — Individual thumbnails (1K–4K resolution)
- **HTML** — Interactive gallery with YouTube mockup cards
- **PDF** — Printable portfolio with stats
- **Strategy Report** — Full HTML document with original analysis, extracted patterns, diagnosed issues, proposed directions with proof videos, and packaging psychology breakdown
- **Before & After** — Side-by-side export

![Before After](./docs/packaging-reviver/18-before-after-side-by-side.png)

---

## Technical Architecture

### Stack

| Layer | Tech |
|-------|------|
| **Frontend** | SvelteKit 2.50+ with Svelte 5 runes + TypeScript 5.9 (strict) |
| **Backend** | Netlify Functions (serverless, SvelteKit server routes) |
| **Database** | Neon PostgreSQL (serverless via HTTP) |
| **AI Text** | Gemini 2.5 Flash (seed queries, title gen, relevance scoring, diagnosis, directions) |
| **AI Image Gen** | NanoBanana Pro — gemini-3-pro-image-preview (with 2 fallback models) |
| **AI Vision** | Gemini Multimodal (pattern extraction, CTR scoring, outlier relevance, layer decomposition) |
| **Face Swap** | Replicate API (codeplugtech/face-swap) with exponential backoff + polling |
| **Image Storage** | Cloudflare R2 (creator portraits, uploaded images) |
| **Image Processing** | @imgly/background-removal (WASM, client-side) |
| **Auth** | Magic-link passwordless (JWT HS256 + HttpOnly cookies, 30-day expiry) |
| **Email** | Resend API (magic link delivery) |
| **Encryption** | AES-256-GCM for API keys at rest (Node crypto) |
| **Local Persistence** | IndexedDB (binary asset cache) |
| **Monorepo** | Turborepo with shared @mo-yaz/shared-auth package |
| **Build** | Vite 7.3+, SvelteKit adapter-netlify |
| **Testing** | Vitest (unit), Playwright (e2e) |

### Data Flow

```
User pastes flopped video URL
    ↓
YouTube API pulls video metadata + channel stats
    ↓
User pastes transcript → Gemini extracts 20 key topics
    ↓
Gemini generates 30 script-informed seed queries
    ↓
Round 1: Snowball searches YouTube across all queries
    ↓
3-layer outlier detection (hard reject → soft flags → AI relevance)
    ↓
Round 2: Snowball refines queries from Round 1 winners
    ↓
Pattern extraction: title formulas + thumbnail visual patterns
    ↓
Strategy: Diagnosis with Change Dial (underperformance ratio → edit intensity)
    ↓
Preserving / Fixing / Recommended Edit tags with evidence
    ↓
Title pattern analysis + packaging direction generation
    ↓
Canvas editor: compose layers from decomposed outlier elements
    ↓
Generate thumbnails via NanoBanana (calibrated by Change Dial + blueprint)
    ↓
Optional: Face swap via Replicate, Relight via 3D sphere UI
    ↓
Vision model scores each thumbnail on 8 dimensions
    ↓
Side-by-side comparison with CTR predictions
    ↓
Export as thumbnail (1K–4K), mockup, strategy report, or before/after
```

### Key Technical Decisions

- **SvelteKit for V3** — The monolithic approach didn't scale. SvelteKit's server-side rendering, form actions, and layout system provide structure. Svelte 5 runes make state management clean across 4 stores (project, canvas, research, generation).
- **Server-side Gemini proxy** — API keys live server-side only, encrypted AES-256-GCM. Frontend never sees them. All sensitive calls go through server routes. Server decrypts key in memory, forwards request, returns response. Eliminates a whole class of key leaks.
- **Transcript-driven research** — Previous versions only used the title for query generation. Key topic extraction from the transcript makes Snowball queries reflect the actual content, not just the packaging.
- **3-layer outlier detection** — Hard reject filters noise (wrong category, wrong language, too long). Soft flags catch mass-publishers and re-uploads. AI relevance scoring handles the nuance — topical fit + visual composition fit scored by Gemini vision.
- **Change Dial system** — Edit intensity proportional to underperformance. 10% of expected views = aggressive edits. 70% = subtle tweaks. Dial ranges 1–5 with MINIMAL/LOW/MEDIUM/HIGH labels. Each level has explicit preservation targets and identity score thresholds.
- **Canvas layer decomposition** — Any reference thumbnail can be broken into subject, background, text, and object layers with bounding boxes. Auto-compose arranges them. This lets creators remix outlier elements instead of starting from a blank canvas.
- **Multi-model fallback chain** — NanoBanana Pro (primary) → NanoBanana 2 → NanoBanana OG. If a model fails, the next one picks up. Same for JSON parsing: 4-strategy repair chain handles Gemini's malformed responses (raw parse → regex fix → markdown extraction → manual parsing).
- **Face swap with retry logic** — Replicate's face-swap model with exponential backoff for rate limits (429), billing cache lag recovery (402), 2-minute polling window, and pre-swap backup for undo.
- **IndexedDB + Neon hybrid persistence** — Large binary assets (generated thumbnails) stay in IndexedDB for speed. Metadata, version history, channel stats, and project state live in Neon. Channel stats cached with 7-day TTL. Dirty flag tracking with debounced cloud sync.
- **YouTube API quota rotation** — Multiple keys rotate transparently to avoid quota exhaustion during heavy research sessions.
- **Turborepo monorepo** — Shared `@mo-yaz/shared-auth` package handles auth, key management, quota tracking, and types across services.

### Database Schema

```
shared tables:
  users              — UUID id, email, created_at
  api_keys           — encrypted_key + iv per service (youtube|gemini|replicate)
  magic_links        — token, email, 15-min expiry, used_at flag
  quota_state        — per-key exceeded_at flag
  channel_cache      — channel stats with 7-day TTL (indexed on updated_at)

packaging reviver:
  reviver_projects       — video metadata, transcript (5K cap), directions JSON,
                           prefs JSON (changeDialValue, portraits), source video JSON
  reviver_query_results  — queries + results per round per project (indexed)
  reviver_iterations     — every thumbnail version: prompt, canvas snapshot,
                           model, aspect, quality, parent lineage, critique scores,
                           face swap flag, import metadata
  ab_snapshots           — views + CTR estimates at snapshot time
```

### Security & Resilience

- **Magic-link passwordless auth** — No passwords stored. Tokens sent via Resend, 15-minute expiry, single-use.
- **JWT with HttpOnly cookies** — HS256 signed, 30-day expiry. Secure against XSS and CSRF.
- **AES-256-GCM encrypted key storage** — User API keys encrypted at rest in Neon. Decrypted in server memory only during API calls.
- **Server-side proxy for all sensitive calls** — YouTube, Gemini, Replicate calls proxied through server routes. Keys never touch the frontend.
- **Ownership verification** — Users can only access, modify, or delete their own keys and projects.
- **Face swap rate limit recovery** — Exponential backoff with 5 retries and cumulative delays. Billing cache lag (402) handled with delayed retry.
- **4-strategy JSON repair chain** — Malformed Gemini responses don't crash. Fallback parsing keeps the pipeline running.
- **50-state undo/redo** — Full canvas history with serialization. No destructive edits.
- **Version history immutability** — All thumbnail iterations saved in Neon with full lineage. Revert or branch from any version.
- **IndexedDB offline caching** — Generated images stay local until synced. Network failures don't lose work-in-progress.
- **Debounced cloud sync** — 2-second debounce on project state saves to prevent write storms.

### API Routes (24 endpoints)

```
Auth:      /api/auth/send-link, /verify, /me, /logout, /account
Projects:  /api/projects, /projects/[id], /projects/[id]/results
AI:        /api/gemini, /api/replicate, /api/decompose, /api/sessions/diagnosis
YouTube:   /api/youtube, /api/channels/[id], /api/transcript, /api/quota
Keys:      /api/keys, /api/keys/sync, /api/settings
Media:     /api/proxy-image, /api/upload-image
System:    /api/sessions/sync, /api/init-db
```

### Project Structure

```
packaging-reviver/
├── src/
│   ├── routes/                # SvelteKit pages — 5-screen flow + API
│   │   ├── api/               # 24 server routes (auth, gemini, youtube, keys, projects)
│   │   ├── research/          # Snowball research flow (queries, outliers, rounds)
│   │   ├── strategy/          # Diagnosis + title analysis + packaging directions
│   │   ├── generate/          # Canvas editor + AI thumbnail generation + face swap
│   │   ├── compare/           # Side-by-side comparison + scoring + export
│   │   ├── auth/              # Magic link login
│   │   └── settings/          # API key management + quota monitoring
│   └── lib/
│       ├── components/        # StepDots, Canvas, QueryChip, OutlierCard, RelightModal, etc.
│       ├── stores/            # 4 Svelte stores (project, canvas, research, generation)
│       ├── services/          # Core logic (youtube, gemini, outlier-engine, replicate,
│       │                      #   canvas-export, layer-service, auto-compose, strategy-report,
│       │                      #   key-manager, gemini-face)
│       ├── server/            # Server-side auth, encryption, DB queries
│       └── utils/             # Helpers, types, constants
├── packages/
│   └── shared-auth/           # @mo-yaz/shared-auth (auth, keys, quota, types)
├── schema.sql                 # PostgreSQL schema (Neon)
├── turbo.json                 # Turborepo config
├── svelte.config.js
├── vite.config.ts
├── package.json
└── netlify.toml
```

---

## Try It

The app is free to use but runs on your own API keys. After signing in with a magic link, go to Settings and add:

1. **Gemini API key** — [Google AI Studio](https://aistudio.google.com/apikey). Used for diagnosis, title generation, packaging directions, thumbnail generation, relevance scoring, and layer decomposition. Free tier is generous.

2. **YouTube Data API v3 key** — [Google Cloud Console](https://console.cloud.google.com/apis/credentials). Create a project, enable YouTube Data API v3, generate a key. Used to pull video stats and run Snowball research. Free tier gives 10,000 quota units/day.

3. **Replicate API key** (optional) — [Replicate](https://replicate.com). Only needed for face swap. Pay-per-use.

Your keys are encrypted server-side (AES-256-GCM) before storage. They never touch the frontend.

---

## About the Author

**Mohamed** — Dubai-based YouTube creator and content strategist. My channel revolves around real-life challenges that reveal the true Dubai. I'm not a developer by training — I build tools because I need them, then clean them up enough to ship publicly.

SecondTake.ai started as a private workflow. It's the system I use to evaluate whether a video concept is packaged well enough to click on.

- [YouTube](https://youtube.com/@mohamed_yaz)
- [LinkedIn](https://linkedin.com/in/momaurane)
- [GitHub](https://github.com/momaurane)
