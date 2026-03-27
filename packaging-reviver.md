# Packaging Reviver

> Rescue flopped videos by data-backed redesign. Paste a YouTube link → runs research pipeline → identifies winning patterns → auto-decomposes thumbnail layers → generates new options → scores against originals.

![Screenshot](./docs/screenshot.png)

## Why This Exists

Video packaging—title, thumbnail, hooks—is responsible for 80% of view count variance. A video that flopped isn't always bad content; it's often bad packaging.

This is the third evolution of the research-and-generate tools. **Version 1** was a monolithic HTML file (YouTube Outlier Research). **Version 2** added thumbnail generation (Thumbnail Studio). **Version 3** (this) unified them into a proper SvelteKit app with server-side key management, type-safe state, and a clear 5-screen user journey.

The pipeline works like this: paste a flopped video → Snowball research finds what wins in that niche → vision analysis identifies thumbnail patterns from winners → decomposes your original thumbnail into layers (background, text, graphics, brand) → "Change Dial" system calibrates edit intensity based on underperformance ratio → Gemini generates polished new thumbnails → side-by-side comparison with CTR prediction.

Instead of guessing what to change, creators get data-backed direction: "Increase contrast by 40%", "Reposition text to upper-left", "Use warmer colors."

## Technical Architecture

### Stack

| Layer | Tech |
|-------|------|
| **Frontend** | SvelteKit with Svelte 5 runes + TypeScript |
| **Backend** | Netlify Edge Functions (SvelteKit server routes) |
| **Database** | Neon PostgreSQL (projects, research cache, versions) |
| **AI Query Gen** | Gemini 2.5 Flash (research queries) |
| **AI Image Gen** | Gemini 2.0 (thumbnail generation) |
| **AI Vision** | Gemini Multimodal (pattern extraction, CTR scoring) |
| **Image Processing** | @imgly/background-removal (WASM, client-side) |
| **Auth** | JWT with HttpOnly cookies |
| **Encryption** | AES-256-GCM for API keys |
| **Persistence** | IndexedDB (local cache) + Neon (persistent) |

### Data Flow

```
User pastes flopped video URL
    ↓
Extract video metadata (YouTube API)
    ↓
Run Snowball research in that niche
    ↓
Vision analysis on winner thumbnails
    ↓
Extract patterns (color palettes, text positions, imagery style)
    ↓
Decompose user's original thumbnail into layers
    ↓
"Change Dial" calibrates intensity (underperformance ratio → edit magnitude)
    ↓
Generate 4 new thumbnail options via Gemini
    ↓
Vision model scores each new thumbnail + original
    ↓
Side-by-side comparison with CTR predictions
    ↓
User previews changes, decides to apply
```

### Key Technical Decisions

- **SvelteKit for v3**: The monolithic approach didn't scale. SvelteKit's server-side rendering, form actions, and layout system provide structure. Svelte 5 runes make state management clean.
- **Server-side Gemini proxy**: API keys live server-side only. Frontend never sees them. All sensitive API calls (Gemini, YouTube) go through server routes. This eliminates a whole class of leaks.
- **Client-side WASM image processing**: @imgly/background-removal runs in the browser—no server round-trip. Deconstructing thumbnails into layers happens locally for instant feedback.
- **"Change Dial" system**: Instead of "generate random variations," we calibrate edit intensity proportionally. If your video got 10% of expected views, we edit more aggressively. If 70%, subtle tweaks. Dial ranges 0-100.
- **4-strategy JSON repair chain**: Gemini sometimes returns malformed JSON (trailing commas, missing quotes). We have a fallback chain: (1) JSON.parse raw, (2) regex fix common errors, (3) extract JSON from markdown blocks, (4) manual parsing. Ensures robustness.
- **IndexedDB + Neon hybrid persistence**: Large binary assets (generated thumbnails) stay in IndexedDB for speed. Metadata and version history live in Neon.

## Security & Resilience

- **Magic-link passwordless auth**: No passwords. Tokens sent via email.
- **Server-side API key management**: Gemini, YouTube, and Resend keys live on the server. Frontend never touches them.
- **AES-256-GCM encrypted key storage**: API keys encrypted in the database with a derived key from the user's magic-link secret.
- **JWT with HttpOnly cookies**: Session tokens are secure against XSS and CSRF.
- **YouTube API quota rotation**: If researching in a niche requires multiple searches, keys rotate to avoid quota exhaustion.
- **4-strategy JSON repair chain**: Malformed Gemini responses don't crash. Fallback parsing keeps the pipeline running.
- **Version history immutability**: All thumbnail versions saved in Neon. Users can revert changes or compare iterations.
- **IndexedDB offline caching**: Generated images stay local until synced. Network failures don't lose work-in-progress.

## Project Structure

```
packaging-reviver/
├── src/
│   ├── app.html
│   ├── routes/
│   │   ├── +page.svelte        # Home / input screen
│   │   ├── +page.server.js    # Input form action
│   │   ├── research/
│   │   │   ├── +page.svelte    # Research results screen
│   │   │   └── +page.server.js # Snowball search orchestration
│   │   ├── strategy/
│   │   │   ├── +page.svelte    # Pattern extraction + Change Dial
│   │   │   └── +page.server.js # Vision analysis of winners
│   │   ├── generate/
│   │   │   ├── +page.svelte    # Gemini generation + local preview
│   │   │   └── +page.server.js # Image generation API
│   │   ├── compare/
│   │   │   ├── +page.svelte    # Side-by-side + CTR scoring
│   │   │   └── +page.server.js # Vision scoring
│   │   ├── projects/
│   │   │   ├── +page.svelte    # Browse saved redesigns
│   │   │   └── +page.server.js # Project CRUD
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   │   ├── +page.svelte
│   │   │   │   └── +page.server.js # Magic-link send
│   │   │   └── verify/
│   │   │       └── +page.server.js # Token verification
│   ├── lib/
│   │   ├── stores/
│   │   │   ├── user.ts         # Auth state (Svelte 5 rune store)
│   │   │   ├── research.ts     # Current research
│   │   │   ├── strategy.ts     # Pattern extraction + Change Dial
│   │   │   └── thumbnails.ts   # Generated versions
│   │   ├── api/
│   │   │   ├── youtube.server.ts
│   │   │   ├── gemini.server.ts
│   │   │   └── db.server.ts
│   │   ├── components/
│   │   │   ├── ResearchPanel.svelte
│   │   │   ├── ChangeDial.svelte
│   │   │   ├── Canvas.svelte
│   │   │   ├── ComparisonCard.svelte
│   │   │   └── ProjectCard.svelte
│   │   ├── utils/
│   │   │   ├── crypto.ts       # AES-256-GCM
│   │   │   ├── indexeddb.ts    # Binary caching
│   │   │   ├── json-repair.ts  # 4-strategy LLM output fix
│   │   │   └── scoring.ts      # CTR prediction
│   │   └── types/
│   │       └── index.ts        # TypeScript interfaces
│   └── styles/
│       └── main.css
├── .env.example
├── .env.local.example
├── svelte.config.js
├── vite.config.js
├── package.json
└── README.md
```

## About the Author

**Mohamed** — Dubai-based YouTube creator and content strategist. My channel revolves around real-life challenges that reveal the true Dubai. I'm not a developer by training — I build tools because I need them, then clean them up enough to ship publicly.

Packaging Reviver started as a private workflow. It's now the system I use to evaluate whether a video concept is packaged well enough to click on.

- [YouTube](https://youtube.com/@mohamed_yaz)
- [LinkedIn](https://linkedin.com/in/momaurane)
- [GitHub](https://github.com/momaurane)
