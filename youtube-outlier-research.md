# YouTube Outlier Research

> Multi-round snowball research that finds what actually works on YouTube. Paste a topic, let the algorithm hunt for statistical outliers, then iterate through refined queries to surface winning content patterns.

![Screenshot](./docs/screenshot.png)

## Why This Exists

YouTube's success formula is hidden in the outliers — videos that perform 2x+ better than the channel average. Most creators chase trending topics without understanding *why* certain videos work. This tool was built in 3 days as an internal research pipeline before creating content.

The MVP solved a real problem: manually checking 50+ YouTube videos to find patterns is tedious and prone to confirmation bias. So I built a script that generated smart queries, searched systematically, flagged outliers, and extracted strategy insights. It worked. The script grew into a full SPA with multi-round refinement, API key rotation, and encrypted storage.

Today it's the research engine behind content decisions. Instead of guessing, you get data-backed insights: which titles resonate, which thumbnail patterns stick, which hooks work.

## Technical Architecture

### Stack

| Layer | Tech |
|-------|------|
| **Frontend** | Vanilla JS SPA (~3,400 lines) + HTML5 Canvas |
| **Backend** | Netlify Functions (Node.js) |
| **Database** | Neon PostgreSQL (serverless) |
| **AI** | Gemini 2.5 Flash (query generation + thumbnail vision analysis) |
| **External APIs** | YouTube Data API v3 (quota rotation across 8 keys) |
| **Auth** | Magic-link via Resend, JWT with HttpOnly cookies |
| **Encryption** | AES-256-GCM for API key storage |

### Data Flow

```
User Input (topic)
    ↓
Gemini 2.5 Flash generates 8 diverse search queries
    ↓
YouTube API searches videos in batches (quota-aware)
    ↓
Algorithm flags outliers (views > 2x channel average)
    ↓
Vision analysis on thumbnails (Gemini multimodal)
    ↓
Pattern extraction (titles, CTAs, color schemes)
    ↓
Refine queries based on winning patterns
    ↓
Repeat rounds 2-3 for deeper research
    ↓
Deliver top outliers with strategy insights
```

### Key Technical Decisions

- **Vanilla JS for v1**: No build step needed. Fast iteration. The 3-day MVP required speed over structure.
- **Gemini 2.5 Flash for query generation**: Small, fast, and accurate at producing diverse search queries that probe different audience angles.
- **Multi-key YouTube API quota rotation**: YouTube limits 10,000 quota units per day per key. With 8 API keys rotated strategically, we can search 8x more videos in a single session.
- **Client-side encrypted key storage**: API keys are AES-256-GCM encrypted with a derived key from the user's magic-link secret. Keys never touch unencrypted databases.
- **Multimodal vision analysis**: Gemini's image understanding lets us extract thumbnail design patterns programmatically instead of manual categorization.
- **2-round minimum refinement**: First round finds breadth, second round refines based on patterns found in round 1. A third round catches nuance.

## Security & Resilience

- **Magic-link passwordless auth**: No passwords stored. Each login generates a short-lived token sent via Resend email.
- **JWT with HttpOnly cookies**: Session tokens are HttpOnly, Secure, and SameSite=Strict. XSS can't steal them.
- **AES-256-GCM encrypted API key storage**: YouTube API keys are encrypted client-side with a key derived from the user's magic-link secret. Stored encrypted in the database.
- **YouTube API quota rotation**: 8 API keys are cycled across searches. If one key hits quota, the next search uses a different key. Tracks usage per key in real time.
- **Graceful API failure**: If Gemini is slow or quota is exhausted, the UI queues the search and retries with exponential backoff.
- **Search result caching**: Recent searches are cached in IndexedDB for offline access and to reduce redundant API calls.

## Project Structure

```
youtube-outlier-research/
├── src/
│   ├── index.html
│   ├── main.js              # App entry point
│   ├── components/
│   │   ├── SearchForm.js    # Topic input + research controls
│   │   ├── ResultsGrid.js   # Outlier display + sorting
│   │   └── PatternPanel.js  # Extracted strategy insights
│   ├── lib/
│   │   ├── api.js           # API wrapper (YouTube, Gemini)
│   │   ├── crypto.js        # AES-256-GCM encryption
│   │   ├── quota.js         # YouTube API key rotation logic
│   │   └── cache.js         # IndexedDB caching
│   └── styles/
│       └── main.css
├── netlify/functions/
│   ├── search.js            # YouTube API search orchestration
│   ├── refine.js            # Pattern extraction + query refinement
│   ├── auth/
│   │   ├── login.js         # Magic-link send
│   │   └── verify.js        # Token verification
│   └── keys.js              # Encrypted key storage
├── .env.example
├── vite.config.js
├── package.json
└── README.md
```

## About the Author

**Mohamed** — Dubai-based YouTube creator and content strategist. My channel revolves around real-life challenges that reveal the true Dubai. I'm not a developer by training — I taught myself to build these tools because the research workflows I needed didn't exist.

This tool runs my actual content pipeline. It's not a side project, it's infrastructure.

- [YouTube](https://youtube.com/@mohamed_yaz)
- [LinkedIn](https://linkedin.com/in/momaurane)
- [GitHub](https://github.com/momaurane)
