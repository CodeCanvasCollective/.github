<div align="center">

# Job Compass — System Architecture

**A polyglot monorepo powering an AI-driven job hunt platform**

</div>

---

## High-Level Architecture

```mermaid
graph TB
    subgraph "User Interface Layer"
        EXT["🧩 Chrome Extension<br/>(Manifest V3 · Vanilla JS)"]
        DESK["🖥️ Electron Desktop App<br/>(Win · macOS · Linux)"]
        WEB["🌐 Marketing Website<br/>(Static HTML/CSS)"]
    end

    subgraph "Content Extraction — 12+ Job Boards"
        LI["LinkedIn"]
        IND["Indeed"]
        CVL["CV-Library"]
        GLASS["Glassdoor"]
        MORE["Seek · ZipRecruiter · Monster<br/>Reed · Naukri · Bayt · Kariyer · Jora"]
    end

    EXT -->|"Content Script<br/>Auto-Extract"| LI
    EXT -->|"Content Script"| IND
    EXT -->|"Content Script"| CVL
    EXT -->|"Content Script"| GLASS
    EXT -->|"Content Script"| MORE

    subgraph "Backend Services (Python 3.13)"
        API["⚡ FastAPI Bridge Server<br/>localhost:8765<br/>CORS Hardened · Rate Limited (60/min)"]

        subgraph "MCP Servers — Model Context Protocol"
            MCP_API["🤖 MCP API Server<br/>Cover Letter Generation<br/>Company Research<br/>Notion Sync"]
            MCP_CLI["🤖 MCP CLI Server<br/>Batch Cover Letters<br/>Skill Extraction & Refinement<br/>Job Analysis & Scoring"]
            MCP_SEARCH["🔍 MCP Search Server<br/>Daily Automated Job Hunt<br/>Salary Insights"]
        end
    end

    EXT -->|"HTTP REST"| API
    DESK -->|"Spawns & Manages<br/>Process Lifecycle"| API
    API --> MCP_API
    API --> MCP_CLI
    API --> MCP_SEARCH

    subgraph "Data Layer"
        DB[("💾 SQLite + WAL Mode<br/>─────────<br/>candidates · jobs<br/>interviews · tech_skills<br/>cover_letter_config")]
        ESTORE["🔐 Encrypted Store<br/>(electron-store)<br/>API Keys · Preferences"]
    end

    API --> DB
    MCP_CLI --> DB
    MCP_SEARCH --> DB
    DESK --> ESTORE

    subgraph "External APIs & Integrations"
        GEMINI["🧠 Google Gemini API<br/>(gemini-2.5-flash)"]
        GEMINI_CLI["🧠 Gemini CLI<br/>(Local Subprocess · 4 Workers)"]
        NOTION["📓 Notion API<br/>(Database Sync · 3 req/sec)"]
        ADZUNA["🔎 Adzuna API<br/>(UK Jobs · 250 req/day)"]
        JOOBLE["🔎 Jooble API<br/>(EU Jobs · 500 req/day)"]
    end

    MCP_API --> GEMINI
    MCP_API --> NOTION
    MCP_CLI --> GEMINI_CLI
    MCP_SEARCH --> ADZUNA
    MCP_SEARCH --> JOOBLE

    subgraph "CI/CD & Distribution"
        GHA["⚙️ GitHub Actions"]
        CWS["Chrome Web Store"]
        R2["☁️ Cloudflare R2<br/>(Desktop Installers)"]
        GHR["📦 GitHub Releases"]
    end

    GHA -->|"Package Extension"| CWS
    GHA -->|"Build Installers<br/>Win · macOS · Linux"| R2
    GHA --> GHR
```

---

## Component Breakdown

### 1. Chrome Extension (Manifest V3)

| Aspect | Detail |
| :--- | :--- |
| **Popup** | Job save & match analysis UI |
| **Content Scripts** | Auto-extract job data from 12+ platforms |
| **Options Dashboard** | 8-tab dashboard (Jobs, Skills, Cover Letters, Interviews, Profile, MCP, Settings) |
| **Background** | Service worker for message routing |
| **Charts** | Chart.js v4.5.1 (CSP-compliant local bundle) |
| **Storage** | Chrome Storage API (local) |

### 2. Electron Desktop App

| Aspect | Detail |
| :--- | :--- |
| **Runtime** | Electron 40.6.1 |
| **Server Manager** | Spawns FastAPI backend, health checks, environment setup |
| **System Tray** | Background operation with tray icon |
| **Auto-Update** | Built-in update mechanism |
| **Security** | IPC preload bridge, encrypted config store |
| **Build Targets** | Windows (NSIS), macOS (DMG arm64 + x64), Linux (AppImage) |

### 3. FastAPI Bridge Server

| Aspect | Detail |
| :--- | :--- |
| **Framework** | FastAPI 0.115 + Uvicorn (async) |
| **Port** | 8765 (localhost) |
| **Routers** | Jobs, Skills, Interviews, Stats, Settings, API Keys, Health |
| **Security** | CORS hardened (localhost + chrome-extension origins only), rate limiting (60/min) |
| **Features** | Duplicate detection (fuzzy + exact), PDF generation, ICS export |

### 4. MCP Servers (Model Context Protocol)

| Server | Tools | External API |
| :--- | :--- | :--- |
| **MCP API** | `generate_cover_letter`, `research_company`, `sync_to_notion`, `check_status` | Gemini API, Notion |
| **MCP CLI** | `refine_job_tech_skills`, `analyze_and_decide`, `generate_cover_letters` (batch), `analyze_skill_gap` | Gemini CLI (subprocess, 4 workers) |
| **MCP Search** | `search_run_daily_job_hunt`, `search_find_matching_jobs`, `search_salary_insights` | Adzuna, Jooble |

---

## Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | Chrome Extension (MV3), Vanilla JS, Chart.js |
| **Desktop** | Electron 40, electron-builder |
| **Backend** | Python 3.13, FastAPI, Uvicorn |
| **AI/ML** | Google Gemini API, Model Context Protocol (MCP) |
| **Database** | SQLite (WAL mode), Encrypted electron-store |
| **Job APIs** | Adzuna, Jooble |
| **Integrations** | Notion API, ICS calendar export |
| **Monorepo** | pnpm + uv workspaces, Turborepo 2.4 |
| **CI/CD** | GitHub Actions, Cloudflare R2 |
| **Testing** | pytest, Playwright E2E |
| **Platforms** | Windows, macOS (Intel + ARM), Linux, Chromium browsers |

---

## Security Model

- **API Keys** — Encrypted at rest via `cryptography` + electron-store
- **CORS** — Hardened origins (localhost, chrome-extension://, job board domains only)
- **Rate Limiting** — 60 requests/minute via slowapi
- **Content Security Policy** — Extension CSP with no inline scripts
- **IPC** — Electron preload bridge (no direct Node.js access from renderer)
- **Data Privacy** — All data stored locally (SQLite), no cloud telemetry

---

<div align="center">

[Back to Organization Profile](../README.md)

</div>
