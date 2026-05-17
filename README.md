# ORBIT: RAG Content Engine v3
### *Intelligence in Motion*

> Multi-agent AI system that ingests research documents and generates platform-ready social media content, with branded image rendering, live web intelligence, media monitoring, and per-section regeneration with custom prompts.

[![Python](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Latest-009688.svg)](https://fastapi.tiangolo.com/)
[![CrewAI](https://img.shields.io/badge/CrewAI-Latest-orange.svg)](https://crewai.com/)
[![Claude](https://img.shields.io/badge/Claude-Sonnet%204.6-8A2BE2.svg)](https://anthropic.com/)
[![React](https://img.shields.io/badge/React-19-61DAFB.svg)](https://react.dev/)
[![Qdrant](https://img.shields.io/badge/Qdrant-Embedded-DC143C.svg)](https://qdrant.tech/)

---

## What Is This?

**ORBIT** is a production-grade multi-agent RAG system built for a client: a leading industry body that publishes research reports, surveys, and policy papers, and needs to turn that research into social media content consistently and at scale.

You feed it documents. It builds a searchable knowledge base. Specialized AI agents (called **Satellites**) retrieve the most relevant data from the right sources and write platform-native content grounded in your actual research. No hallucinated statistics. Every claim in Research Findings carries a source reference (document name, year, page number).

Three modes:

- **Knowledge Base**: Generate content from your document library on any topic, any platform combination, with filtering down to specific reports
- **News Intelligence**: React to current events by combining live Perplexity web intelligence with your historical research data in a three-phase pipeline
- **Media Monitoring**: Scan live news for saved keywords within any date range, get 5 structured story briefings with 13 intelligence fields and a one-click "Use This" that fills the NI form

Built by my agency for the client. Now open-sourced as a reference implementation for organizations with research libraries that deserve a wider audience.

---

## Screenshots

| | | |
|---|---|---|
| ![1](ss/1.png) | ![2](ss/2.png) | ![3](ss/3.png) |
| ![4](ss/4.png) | ![5](ss/5.png) | ![6](ss/6.png) |
| ![7](ss/7.png) | ![8](ss/8.png) | ![9](ss/9.png) |
| ![10](ss/10.png) | ![11](ss/11.png) | ![12](ss/12.png) |
| ![13](ss/13.png) | ![14](ss/14.png) | ![15](ss/15.png) |
| ![16](ss/16.png) | ![17](ss/17.png) | ![18](ss/18.png) |
| ![19](ss/19.png) | ![20](ss/20.png) | ![21](ss/21.png) |
| ![22](ss/22.png) | ![23](ss/23.png) | ![24](ss/24.png) |
| ![25](ss/25.png) | ![26](ss/26.png) | ![27](ss/27.png) |
| ![28](ss/28.png) | ![29](ss/29.png) | ![30](ss/30.png) |
| ![31](ss/31.png) | ![32](ss/32.png) | ![33](ss/33.png) |
| ![34](ss/34.png) | ![35](ss/35.png) | ![36](ss/36.png) |
| ![37](ss/37.png) | ![38](ss/38.png) | ![39](ss/39.png) |
| ![40](ss/40.png) | ![41](ss/41.png) | ![42](ss/42.png) |

---

## How It Works

```
Research PDFs  →  Qdrant Vector DB  →  CrewAI Agents  →  Platform Content
                   (hybrid search)    (Claude Sonnet 4.6)   LinkedIn / Instagram
                        +                                    YouTube / X / Facebook
               Cross-encoder rerank
                              ↑
                  Perplexity Web Search (News Intelligence + Media Monitoring)
                              ↓
                  PNG Image Renderer (Playwright / Chromium)
```

1. **Ingestion**: PDFs parsed with pdfplumber + unstructured, semantically chunked, embedded with OpenAI `text-embedding-3-large` (3072-dim), stored in Qdrant (one collection per domain)
2. **Retrieval**: Hybrid dense vector + BM25 keyword search with cross-encoder re-ranking. Top-6 most relevant passages reach the agent. Low-relevance results filtered before the LLM ever sees them
3. **Knowledge Base generation**: Sequential CrewAI crew per domain: research task → content writing → conditional creative direction. Multiple domains run in sequence with rate-limit-safe inter-domain delays; combined research passes to one unified writing task
4. **News Intelligence pipeline**: Three phases: (1) Perplexity web search for live context; (2) per-domain RAG research with web context injected; (3) synthesis of all domain findings into one unified analyst brief, then content writing from that synthesis
5. **Media Monitoring**: Perplexity + Qdrant historical query run in parallel. Claude Haiku generates 5 story briefings with 13 intelligence fields in one pass. Results persist to SQLite until next scan
6. **Regeneration**: Any completed section (Research Findings or any platform card) can be regenerated independently. Custom prompt steers style. Same report filters from original generation applied automatically. Server-side injection blocking prevents prompt bypass
7. **Image rendering**: Carousel slides (1080×1350), X cards (1200×675), and Facebook slides (1080×1080) rendered as branded PNGs using Playwright headless Chromium: 6 CSS chart types, no JS libraries

---

## Three Modes

### Knowledge Base
Generate content from your document library on any topic and platform combination.

- Pick one or more research domains
- **Report Selection Popup**: before confirming each domain, filter down to specific reports. Domain card shows a teal badge ("All reports" or "X of Y reports")
- Three-layer citation enforcement when a report filter is active (agent backstory injection, task-level restriction block, and post-processing `_filter_citations()`) because LLM training knowledge is strong enough to override a single instruction layer
- Multi-domain: each domain runs its own RAG research pass, combined research feeds one unified writing task
- Expected time: ~3 minutes

### News Intelligence
React to breaking events with analyst-style content backed by years of documented data.

- Describe the event (up to 500 characters)
- Select relevant research domains with optional per-report filtering
- Three-phase pipeline: web search → per-domain RAG → synthesis + content writing
- Research Findings includes Perplexity source URLs as bullet points (live web citations separate from document citations)
- Expected time: 6-8 minutes

### Media Monitoring
Stay on top of relevant news. Turn any story into an NI prompt in one click.

- Save keywords, stored permanently in SQLite
- Run scans for any date range (7 days to custom)
- 5 structured story briefings per scan: headline, 4-5 sentence summary, sentiment pill, and Full Analysis
- Full Analysis overlay: all 13 intelligence fields + a suggested NI prompt
- "Use This" fills the NI event textarea and navigates to NI directly
- NI sidebar shows last MM results as prompt cards, closed by default

---

## What It Outputs

| Platform | Output |
|---|---|
| **LinkedIn** | 1,500–1,800 char thought-leadership post: bold hook, data bullets, analyst insight layer, closing question, hashtags |
| **X (Twitter)** | Lead tweet + 5–7 tweet thread + 3 comment variations + rendered X card PNG (1200×675) |
| **Instagram Carousel** | 10-slide story arc with LLM-chosen highlight phrase per slide and optional data chart. All 10 slides rendered as branded PNGs (1080×1350) |
| **Instagram Reels** | 6-10 sec video concept: hook, on-screen text, voiceover direction, visual concept |
| **YouTube** | Full spoken script (400-500 words) with [HOOK] [CONTEXT] [DATA DEEP-DIVE] [SO WHAT] [CTA]: word counts enforced per section |
| **Facebook** | 3-paragraph analytical caption + hashtags + 4 branded square slide images (1080×1080): cover, data/chart, insight, CTA |

**Creative direction** (Visual Design and Video & Audio briefs) generated as glass-style accordion dropdowns inside each relevant platform card.

---

## Key Features

- **Anti-hallucination**: Agents use only RAG-retrieved data. Research Findings carry full citations (document name, year, page number). Platform content references the source naturally: no academic citation format in posts
- **Regeneration with custom prompt**: Reruns any section independently. Optional custom instruction steers style or focus. New version persisted to DB: refresh always shows latest
- **Report filtering**: Per-domain per-report filtering. Three-layer enforcement ensures the LLM stays within the selected reports even when it has strong training knowledge about the source material
- **Robust JSON resilience**: If LLM output is malformed or truncated, the frontend salvage parser extracts each platform's JSON block individually: no platform card goes blank
- **Soft cancellation**: Cancel any active generation from the Status page. Background thread finishes silently without blocking new requests
- **Persistent history**: All tasks stored in SQLite. Survives server restarts. H/T badges, search, delete with confirmation
- **Rate limit handling**: Exponential backoff on Anthropic 429 errors (30s → 60s → 120s → 240s). Inter-domain delays prevent hitting the token-per-minute limit
- **Galaxy UI**: Animated deep-space theme, orbital loader, semi-transparent glass sidebar, platform-coloured card headers, fullscreen lightbox with keyboard navigation

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Agent Framework** | CrewAI |
| **LLM** | Claude Sonnet 4.6 (Anthropic), `max_tokens 16384` |
| **Vector DB** | Qdrant (embedded local / cloud) |
| **Embeddings** | OpenAI text-embedding-3-large (3072-dim) |
| **Re-ranking** | cross-encoder/ms-marco-MiniLM-L-6-v2 |
| **Web Search** | Perplexity sonar-pro |
| **Image Rendering** | Playwright + headless Chromium |
| **Backend** | FastAPI + Python 3.11+ |
| **Frontend** | React 19 + Vite + Tailwind CSS 4 |
| **Database** | SQLite (tasks + keywords, persistent) |
| **Auth** | JWT (24h session) |
| **PDF Parsing** | pdfplumber + unstructured |

---

## Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+
- API keys: Anthropic (LLM), OpenAI (embeddings), Perplexity (web search)
- Playwright Chromium (image generation)

### Setup

```bash
# 1. Clone and set up Python environment
git clone https://github.com/adeel-iqbal/orbit-rag-engine.git
cd orbit-rag-engine
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 2. Install Playwright Chromium
playwright install chromium --with-deps

# 3. Configure environment
cp .env.example .env
# Fill in: OPENAI_API_KEY, ANTHROPIC_API_KEY, PERPLEXITY_API_KEY, AUTH_USERNAME, AUTH_PASSWORD

# 4. Ingest your documents (one-time)
# Place PDFs in data/raw_pdfs/<domain>/ folders, then:
python -m src.ingestion.ingest_all

# 5. Install frontend dependencies
cd frontend && npm install && cd ..

# 6. Run
./run.sh
# Frontend: http://localhost:5173
# Backend:  http://localhost:8000
```

### Environment Variables

| Variable | Required | Purpose |
|---|---|---|
| `OPENAI_API_KEY` | Yes | Embeddings (text-embedding-3-large) |
| `ANTHROPIC_API_KEY` | Yes | LLM: Claude Sonnet 4.6 |
| `PERPLEXITY_API_KEY` | Yes | Live web search |
| `AUTH_USERNAME` | Yes | Login username |
| `AUTH_PASSWORD` | Yes | Login password |
| `QDRANT_MODE` | No | `local` (default) or `cloud` |
| `QDRANT_URL` | No | Qdrant Cloud URL (production) |

---

## Project Structure

```
orbit-rag-engine/
│
├── src/
│   ├── ingestion/          PDF parsing, chunking, embedding, Qdrant upsert
│   ├── agents/             CrewAI agents, RAG tools, brand voice, output schemas
│   │   ├── config/         agents.yaml + tasks.yaml
│   │   └── tools/          rag_tools.py, web_search_tool.py
│   └── api/                FastAPI routes, schemas, middleware
│       ├── routes/         content, reactive, images, keywords, suggestions, auth
│       ├── suggestion_pipeline.py   Media Monitoring pipeline
│       ├── keyword_store.py         SQLite keyword management
│       └── image_generator.py       Playwright PNG renderer
│
├── frontend/               React 19 + Vite + Tailwind CSS 4
│   └── src/
│       ├── pages/          Dashboard, Generate, Reactive, MediaMonitoring,
│       │                   Status, History, Technical, Splash, Login
│       ├── components/     Layout, ContentPreview, PlatformSelector
│       └── context/        AuthContext, ThemeContext
│
├── data/raw_pdfs/          Source documents by domain (8 folders)
├── ss/                     Screenshots
├── requirements.txt
└── run.sh                  One-command startup script
```

---

## Adapting for Your Use Case

ORBIT is a framework, not a fixed product. Swap any layer:

- **Documents**: Point it at your own PDFs in any domain structure
- **LLM**: Claude Sonnet 4.6 by default; model-agnostic via CrewAI
- **Web search**: Perplexity isolated in one file; swap to Brave, Tavily, or any API
- **Platforms**: Add or remove platform cards via modular YAML configs
- **Brand voice**: Reads from a plain text file you can edit
- **Domains**: Add any domain by creating a new Qdrant collection, CrewAI agent, RAG tool entry, and UI card

---

## Deployment

Docker-ready. Tested on Google Cloud Run.

```bash
docker build -t orbit .
docker run -p 8000:8000 --env-file .env orbit
```

For Cloud Run: use `QDRANT_MODE=cloud` with a managed Qdrant Cloud instance. Mount a persistent volume for `orbit_data.db` (SQLite task + keyword history).

```dockerfile
# Required for image generation in container
RUN playwright install chromium --with-deps
```

---

## 📧 Contact

**Adeel Iqbal**

- 📧 Email: [adeelmemon096@yahoo.com](mailto:adeelmemon096@yahoo.com)
- 💼 LinkedIn: [linkedin.com/in/adeeliqbalmemon](https://linkedin.com/in/adeeliqbalmemon)
- 🐙 GitHub: [@adeel-iqbal](https://github.com/adeel-iqbal)

Open to discussions on custom builds, white-labeling, domain-specific deployments, and research-to-content workflow integrations.

---

<div align="center">
  <p>Made with ❤️ by Adeel Iqbal</p>
  <p>⭐ Star this repo if you find it useful!</p>
</div>

---

*Built with CrewAI · Anthropic Claude · Qdrant · Perplexity · Playwright · FastAPI · React*
