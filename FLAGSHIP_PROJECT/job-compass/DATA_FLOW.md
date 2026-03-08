<div align="center">

# Job Compass — Data Flow Diagrams

**How data moves through the system across key workflows**

</div>

---

## 1. Save Job from Job Board

```mermaid
sequenceDiagram
    actor U as User
    participant JB as Job Board<br/>(LinkedIn/Indeed/...)
    participant CS as Content Script
    participant POP as Extension Popup
    participant API as FastAPI Server
    participant DB as SQLite Database

    U->>JB: Opens job listing page
    JB-->>CS: DOM loaded
    CS->>CS: Detect site via hostname
    CS->>CS: Extract job data<br/>(title, company, location,<br/>salary, description, URL)
    CS-->>POP: Send extracted data

    U->>POP: Review & click "Save Job"
    POP->>API: POST /jobs/save_job
    API->>DB: Check duplicates<br/>(exact URL → exact title+company → fuzzy ≥75%)

    alt Duplicate Found
        API-->>POP: ⚠️ Duplicate warning + existing job
    else New Job
        API->>DB: INSERT job record
        API->>DB: Extract & link tech skills
        API-->>POP: ✅ job_id + pending counts
    end

    POP-->>U: Success notification
```

---

## 2. AI Job Match Analysis

```mermaid
sequenceDiagram
    actor U as User
    participant EXT as Extension Popup
    participant API as FastAPI Server
    participant DB as SQLite Database

    U->>EXT: Paste job description<br/>Click "Analyze Match"
    EXT->>API: POST /jobs/analyze_job<br/>{description, title}

    API->>DB: Fetch candidate skills<br/>(candidate_has = true)
    API->>API: Extract tech skills from description<br/>(regex pattern matching)

    API->>API: Calculate score
    Note right of API: +7 per matched skill<br/>+20 senior title bonus<br/>-20 negative keyword penalty<br/>-3 per skill gap

    API-->>EXT: Response
    Note left of EXT: score: 78<br/>rating: "Strong Match"<br/>matched: [Python, FastAPI, ...]<br/>gaps: [Kubernetes, ...]<br/>salary_hint: "£65k-£85k"

    EXT-->>U: Visual score card<br/>with color + emoji
```

---

## 3. Cover Letter Generation

```mermaid
sequenceDiagram
    actor U as User
    participant DASH as Extension Dashboard
    participant API as FastAPI Server
    participant MCP as MCP CLI Server
    participant GEM as Gemini CLI<br/>(subprocess)
    participant DB as SQLite Database

    U->>DASH: Select job → "Generate Cover Letter"
    DASH->>API: Request CL generation

    API->>DB: Fetch candidate profile<br/>(name, skills, experience)
    API->>DB: Fetch job details<br/>(title, company, description)
    API->>DB: Fetch cover letter rules<br/>(tone, format, custom rules)

    API->>MCP: generate_cover_letters tool
    MCP->>GEM: Spawn subprocess with prompt
    Note right of GEM: Inputs:<br/>• Candidate profile<br/>• Job description<br/>• CL rules & template<br/>• Company context

    GEM-->>MCP: Generated cover letter text
    MCP->>DB: Store cover letter on job record
    MCP-->>API: Cover letter response

    API-->>DASH: Cover letter text
    DASH-->>U: Preview with edit option
```

---

## 4. Automated Daily Job Search

```mermaid
sequenceDiagram
    actor U as User
    participant MCP as MCP Search Server
    participant ADZ as Adzuna API<br/>(250/day)
    participant JOB as Jooble API<br/>(500/day)
    participant DB as SQLite Database

    U->>MCP: Trigger daily job hunt<br/>(or scheduled run)

    par Search Adzuna
        MCP->>ADZ: Query by category<br/>(Node.js, .NET, Python, Full Stack)
        ADZ-->>MCP: Job listings batch
    and Search Jooble
        MCP->>JOB: Query by keywords + location
        JOB-->>MCP: Job listings batch
    end

    MCP->>MCP: Normalize results<br/>(title, company, salary, URL, source)

    loop For each job
        MCP->>DB: Check duplicate<br/>(URL match → title+company match)
        alt New Job
            MCP->>DB: INSERT with status "To Apply"
            MCP->>DB: Extract & link tech skills
        else Duplicate
            MCP->>MCP: Skip (log)
        end
    end

    MCP-->>U: Summary<br/>"Found 23 new jobs<br/>(12 Adzuna + 11 Jooble)"
```

---

## 5. Notion Sync

```mermaid
sequenceDiagram
    actor U as User
    participant DASH as Extension Dashboard
    participant API as FastAPI Server
    participant MCP as MCP API Server
    participant NOT as Notion API
    participant DB as SQLite Database

    U->>DASH: Mark job as "Applied"
    DASH->>API: PATCH /jobs/{id}/status
    API->>DB: Update status → "Applied"
    API->>DB: Set pending_notion_sync = true

    Note over MCP: Sync triggered (manual or batch)

    MCP->>DB: Fetch jobs where<br/>pending_notion_sync = true

    loop For each pending job
        MCP->>DB: Get full job details +<br/>cover letter + matched skills
        MCP->>NOT: Create Notion page
        Note right of NOT: Rich text block:<br/>• Title & Company<br/>• Status & Source<br/>• Salary & Location<br/>• Cover Letter<br/>• Matched Skills
        NOT-->>MCP: Page created (page_id)
        MCP->>DB: Set pending_notion_sync = false<br/>Store notion_page_id
    end

    MCP-->>DASH: "Synced 3 jobs to Notion"
```

---

## 6. Interview Pipeline & Reminders

```mermaid
sequenceDiagram
    actor U as User
    participant DASH as Extension Dashboard
    participant API as FastAPI Server
    participant DB as SQLite Database
    participant DESK as Electron Desktop App

    U->>DASH: Schedule interview<br/>(date, type, round, interviewers)
    DASH->>API: POST /interviews/
    API->>DB: INSERT interview record
    API-->>DASH: interview_id

    Note over DASH: User can export to calendar
    U->>DASH: Click "Export .ics"
    DASH->>API: GET /interviews/{id}/ics
    API-->>DASH: .ics file download

    loop Every 60 seconds
        DESK->>API: GET /interviews/upcoming?days=2
        API->>DB: Query interviews within 2 days
        API-->>DESK: Upcoming interviews list

        alt Interview within 30 min & not notified
            DESK->>DESK: Show desktop notification
            Note right of DESK: "Interview in 30 min!<br/>Software Engineer @ Google<br/>Technical Round 2"
            DESK->>DESK: Mark as notified<br/>(encrypted store)
        end
    end
```

---

## 7. Skills Demand & Gap Analysis

```mermaid
sequenceDiagram
    actor U as User
    participant DASH as Extension Dashboard
    participant API as FastAPI Server
    participant MCP as MCP CLI Server
    participant GEM as Gemini CLI
    participant DB as SQLite Database

    U->>DASH: Open Skills tab
    DASH->>API: GET /skills/demand
    API->>DB: Count jobs per tech skill<br/>(demand_count)
    API-->>DASH: Ranked skill demand list

    U->>DASH: Click "Analyze Gap"
    DASH->>API: GET /skills/gap
    API->>DB: Fetch candidate skills<br/>(candidate_has = true)
    API->>DB: Fetch top demanded skills
    API->>API: Compare → identify gaps
    API-->>DASH: Gap report

    Note over DASH: For deeper AI analysis
    U->>DASH: "AI Skill Analysis"
    DASH->>MCP: analyze_skill_gap tool
    MCP->>DB: Fetch full profile + all job skills
    MCP->>GEM: Analyze with Gemini
    GEM-->>MCP: Prioritized learning path
    MCP-->>DASH: Recommendations<br/>"Learn Kubernetes (demanded by 45% of jobs)"
```

---

## Data Storage Summary

```mermaid
erDiagram
    candidate ||--o{ candidate_experiences : has
    candidate ||--o{ candidate_education : has
    candidate ||--o{ candidate_certifications : has
    candidate ||--o{ jobs : tracks
    jobs ||--o{ interviews : schedules
    jobs ||--o{ job_tech_skills : requires
    tech_skills ||--o{ job_tech_skills : "linked to"
    candidate ||--o| cover_letter_config : configures

    candidate {
        int id PK
        string name
        string email
        json skills
        json experience
        string salary_range
    }

    jobs {
        int id PK
        int candidate_id FK
        string title
        string company
        string location
        string salary
        text description
        string status
        text cover_letter
        int relevance_score
        string source
        string apply_url
    }

    interviews {
        int id PK
        int job_id FK
        datetime date
        string type
        int round
        json interviewers
        string outcome
        text feedback
    }

    tech_skills {
        int id PK
        string name
        string category
        boolean candidate_has
        int demand_count
    }
```

### Status Workflow

```mermaid
stateDiagram-v2
    [*] --> ToApply: Job saved
    ToApply --> Apply: Matches profile
    ToApply --> Expired: Listing expired
    Apply --> Applied: Submitted application
    Apply --> PrepareCL: Generate cover letter
    PrepareCL --> Applied: CL ready, submitted
    Applied --> Interview: Got callback
    Applied --> Rejected: No response
    Interview --> Offer: Passed all rounds
    Interview --> Rejected: Did not pass
    Offer --> [*]: Accepted
    Offer --> Withdrawn: Declined offer
    Rejected --> [*]
    Withdrawn --> [*]
    Expired --> [*]
    Applied --> Synced: Notion sync
```

---

<div align="center">

[Back to Organization Profile](../../profile/README.md)

</div>
