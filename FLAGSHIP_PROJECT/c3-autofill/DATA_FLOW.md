<div align="center">

# C3 Autofill - Data Flow Diagrams

**How profile data moves from local storage into live web forms**

</div>

---

## 1. Save or Update a Profile

```mermaid
sequenceDiagram
    actor U as User
    participant OPT as Options (React)
    participant ST as Chrome Storage

    U->>OPT: Enter profile values and form entries
    OPT->>OPT: Validate with isValidProfile / isValidEntry
    OPT->>ST: saveProfiles() / saveEntries()
    ST-->>OPT: Stored successfully
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

    U->>PAGE: Open a form page
    PAGE-->>CS: DOM becomes available
    CS->>CS: Scan labels, names, placeholders, and grouping
    CS->>BG: Request active profile and entries
    BG->>ST: Load profiles, entries, and settings
    ST-->>BG: StorageData (c3Profiles, c3Entries, c3Settings)
    BG-->>CS: Profile entries and fill mode (Overwrite/Append)
    CS-->>U: Ready-to-fill page state
```

---

## 3. Autofill a Form

```mermaid
sequenceDiagram
    actor U as User
    participant POP as Popup (React)
    participant BG as Background Service Worker
    participant CS as Content Script
    participant PAGE as Web Form

    U->>POP: Click "Fill Form"
    POP->>BG: fillForm message (profileId, entries)
    BG->>BG: Decrypt sensitive fields (AES-GCM)
    BG->>CS: Send FormDataEntry[] with decrypted values

    loop For each FormDataEntry
        CS->>CS: Match field by type, name, selector
        CS->>PAGE: Apply value (Overwrite or Append mode)
    end

    CS-->>BG: Fill summary
    BG-->>POP: Result counts
    POP-->>U: Show result summary
```

---

## 4. Export User Data

```mermaid
sequenceDiagram
    actor U as User
    participant OPT as Options (React)
    participant ST as Chrome Storage

    U->>OPT: Click "Export" in Advanced tab
    OPT->>ST: loadAllData()
    ST-->>OPT: StorageData snapshot
    OPT->>OPT: Serialize to JSON
    OPT-->>U: Download backup file
```

---

## 5. Import User Data

```mermaid
sequenceDiagram
    actor U as User
    participant OPT as Options (React)
    participant ST as Chrome Storage

    U->>OPT: Select a JSON backup file
    OPT->>OPT: Parse and validate structure
    OPT->>ST: importData() — merge into storage
    ST-->>OPT: Import complete
    OPT-->>U: Profiles and entries restored
```

---

## 6. Encryption Flow

```mermaid
sequenceDiagram
    participant OPT as Options (React)
    participant CRYPTO as helper.ts (Web Crypto API)
    participant ST as Chrome Storage

    Note over OPT,ST: Saving a sensitive field
    OPT->>CRYPTO: encryptValue(value, cryptoKey)
    CRYPTO->>CRYPTO: AES-GCM encrypt with random IV
    CRYPTO-->>OPT: Base64 encoded ciphertext
    OPT->>ST: Store encrypted value

    Note over OPT,ST: Reading a sensitive field
    ST-->>OPT: Encrypted value
    OPT->>CRYPTO: decryptValue(ciphertext, cryptoKey)
    CRYPTO->>CRYPTO: AES-GCM decrypt
    CRYPTO-->>OPT: Plaintext value
```

---

## Data Storage Model

```mermaid
erDiagram
    StorageData ||--o{ Profile : "c3Profiles"
    StorageData ||--o{ FormDataEntry : "c3Entries"
    StorageData ||--|| Settings : "c3Settings"
    StorageData ||--o{ FakerCategory : "c3FakerData"
    Profile ||--o{ FormDataEntry : "profileId"

    Profile {
        string key PK
        string site
        string hostname
        string createdAt
        string modifiedAt
    }

    FormDataEntry {
        string type
        string field
        string value
        EntryMode mode
        string profileId FK
    }

    Settings {
        boolean enableSensitiveFields
        record sensitiveFields
        boolean encryptSensitive
        boolean autoloadEnabled
        number autoloadDelay
    }

    FakerCategory {
        string id PK
        string label
        array values
        array keywords
    }
```

---

## Autofill Workflow

```mermaid
stateDiagram-v2
    [*] --> ProfileReady: User saves profile and entries
    ProfileReady --> PageDetected: Supported form opened
    PageDetected --> Mapping: Content script scans fields
    Mapping --> Fill: User clicks Fill Form
    Fill --> Decrypt: Sensitive fields detected
    Decrypt --> Apply: AES-GCM decryption
    Fill --> Apply: No sensitive fields
    Apply --> Complete: All entries applied (Overwrite/Append)
    Complete --> [*]
```

---

<div align="center">

[Back to Organization Profile](../../profile/README.md)

</div>
