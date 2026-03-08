<div align="center">

# C3 Autofill - Data Flow Diagrams

**How profile data moves from local storage into live web forms**

</div>

---

## 1. Save or Update a Profile

```mermaid
sequenceDiagram
    actor U as User
    participant OPT as Options Dashboard
    participant BG as Background Service Worker
    participant ST as Chrome Storage

    U->>OPT: Enter profile values and preferences
    OPT->>BG: Save profile payload
    BG->>ST: Persist profile and settings
    ST-->>BG: Stored successfully
    BG-->>OPT: Save confirmation
    OPT-->>U: Updated profile ready for autofill
```

---

## 2. Detect Fields on a Web Page

```mermaid
sequenceDiagram
    actor U as User
    participant PAGE as Web Form
    participant CS as Content Script
    participant BG as Background Service Worker
    participant ST as Chrome Storage

    U->>PAGE: Open a supported form page
    PAGE-->>CS: DOM becomes available
    CS->>CS: Scan labels, names, placeholders, and grouping
    CS->>BG: Request saved profile and rules
    BG->>ST: Load local autofill data
    ST-->>BG: Profile and preferences
    BG-->>CS: Fill candidates and mapping rules
    CS-->>U: Ready-to-fill page state
```

---

## 3. Autofill a Form

```mermaid
sequenceDiagram
    actor U as User
    participant POP as Popup UI
    participant BG as Background Service Worker
    participant CS as Content Script
    participant PAGE as Web Form

    U->>POP: Click "Autofill"
    POP->>BG: Start fill for active tab
    BG->>CS: Send profile data and rules
    CS->>CS: Match fields with confidence scoring

    loop For each matched field
        CS->>PAGE: Apply value to input or select
        CS->>CS: Mark success or skip if ambiguous
    end

    CS-->>BG: Fill summary
    BG-->>POP: Filled, skipped, and review counts
    POP-->>U: Show result summary
```

---

## 4. Export User Data

```mermaid
sequenceDiagram
    actor U as User
    participant OPT as Options Dashboard
    participant BG as Background Service Worker
    participant ST as Chrome Storage

    U->>OPT: Click "Export JSON"
    OPT->>BG: Request export payload
    BG->>ST: Read profiles, rules, and settings
    ST-->>BG: Local data snapshot
    BG-->>OPT: Structured JSON payload
    OPT-->>U: Download backup file
```

---

## 5. Import User Data

```mermaid
sequenceDiagram
    actor U as User
    participant OPT as Options Dashboard
    participant BG as Background Service Worker
    participant ST as Chrome Storage

    U->>OPT: Select an exported JSON file
    OPT->>OPT: Validate file structure
    OPT->>BG: Submit import payload
    BG->>ST: Replace or merge local records
    ST-->>BG: Import saved
    BG-->>OPT: Import status
    OPT-->>U: Profiles and rules restored
```

---

## Data Storage Summary

```mermaid
erDiagram
    profile ||--o{ field_mapping : uses
    profile ||--o{ preference : configures
    export_bundle ||--o{ profile : contains
    export_bundle ||--o{ field_mapping : contains

    profile {
        string id PK
        string name
        json personal_data
        json professional_data
    }

    field_mapping {
        string id PK
        string profile_id FK
        string field_key
        string selector_hint
        string fallback_value
    }

    preference {
        string id PK
        string profile_id FK
        boolean auto_fill_enabled
        boolean confirm_before_fill
    }

    export_bundle {
        string version
        datetime exported_at
        json payload
    }
```

---

## Autofill Workflow

```mermaid
stateDiagram-v2
    [*] --> ProfileReady: User saves profile
    ProfileReady --> PageDetected: Supported form opened
    PageDetected --> Mapping: Extension inspects fields
    Mapping --> Fill: User starts autofill
    Fill --> Review: Some fields skipped or need confirmation
    Fill --> Complete: All mapped fields applied
    Review --> Complete: User confirms or edits values
    Complete --> Exported: User downloads backup
    Exported --> [*]
```

---

<div align="center">

[Back to Organization Profile](../../profile/README.md)

</div>
