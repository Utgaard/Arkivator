# Arkivator – Product Requirements Document

**Version:** 1.0
**Date:** 2026-02-20
**Status:** Draft

---

## 1. Problem Statement

Norwegian households receive a steady stream of paper medical documents — innkallinger (appointment summons), epikriser (discharge/clinical summaries), lab results, referral letters, and prescriptions. These accumulate over time, making it nearly impossible to maintain a coherent, searchable health history for oneself or one's family.

The core pain points:
- Documents pile up physically with no structure or indexing.
- No single overview of the health timeline for family members.
- Sharing relevant history with a new doctor, specialist, or AI assistant is tedious.
- Finding a specific document quickly under stress (e.g., before an appointment) is difficult.
- Original paper may be needed as evidence, but managing both paper and digital copies is cumbersome.

---

## 2. Product Vision

**Arkivator** is an iOS app (iPhone-first) that turns the camera into a medical document inbox. A user photographs a paper document, and Arkivator fully automates the rest: image enhancement, OCR & text extraction, structured titling, semantic tagging, and upload to Google Drive in the correct folder hierarchy — retaining both the cleaned image and a text-readable `.docx` file.

The result is a private, AI-queryable health archive for the whole family, stored in a cloud the user already owns.

---

## 3. Target Users

| User | Description |
|------|-------------|
| **Primary** | Norwegian adults managing their own and/or their children's medical paper trail |
| **Secondary** | Caregivers managing documents for elderly relatives |

### Primary Persona

- Norwegian, 30–55 years old
- iPhone 15 or similar
- Uses Google Drive for personal storage
- Comfortable with apps but not a developer
- Wants zero manual filing work after taking the photo
- May want to paste document text into ChatGPT or another AI to get a plain-language explanation

---

## 4. Goals & Success Metrics

| Goal | Metric |
|------|--------|
| Reduce time to file a document to under 60 seconds | Time-to-file measured in app |
| Achieve ≥95% correct text extraction on standard A4 medical documents | OCR accuracy rate |
| Auto-tag documents correctly ≥90% of the time | Tagging precision |
| Users can retrieve any document within 30 seconds via search | Search latency + user testing |
| Zero documents lost (reliable cloud sync) | Sync failure rate < 0.1% |

---

## 5. Scope

### 5.1 In Scope (v1.0)

- iPhone camera capture with guided shooting UI
- Automatic image enhancement (perspective correction, deskew, brightness/contrast)
- OCR and full-text extraction (Norwegian language primary)
- Structured document title generation: `YYYY-MM-DD <Descriptive Title>`
- Semantic tagging (document type, person, medical specialty, date, institution)
- Google Drive integration: OAuth2 sign-in, folder selection, automatic folder routing
- Dual storage: `.docx` text file + `.jpeg`/`.pdf` image file per document
- Per-person sub-folders (e.g., one folder per family member)
- In-app document list with local search
- Manual edit/correction of extracted text, title, and tags before upload

### 5.2 Out of Scope (v1.0)

- Android support
- Non-Google cloud providers (iCloud, Dropbox) — considered for v2
- Direct AI chat interface within the app (user exports text to preferred AI tool)
- Multi-page document stitching (v1 handles single-sheet photos; v2 target)
- Sharing documents with third parties directly from the app
- FHIR or HelseNorge API integration

---

## 6. User Stories

### Epic 1 – Capture

| ID | Story | Priority |
|----|-------|----------|
| C-1 | As a user, I want to photograph a document and have the app automatically detect and crop the page so I don't need to frame it perfectly. | Must |
| C-2 | As a user, I want the app to enhance the image (remove shadows, correct skew, boost contrast) so the text is clearly readable. | Must |
| C-3 | As a user, I want to retake or manually crop the image before processing if the auto-crop is wrong. | Must |
| C-4 | As a user, I want to capture multiple pages of a multi-page document as a batch. | Should (v1.1) |

### Epic 2 – Extraction & Classification

| ID | Story | Priority |
|----|-------|----------|
| E-1 | As a user, I want the app to extract all text from the document using OCR so I can read and search it digitally. | Must |
| E-2 | As a user, I want the app to automatically generate a title in the format `YYYY-MM-DD <Descriptive Title>` based on the document content. | Must |
| E-3 | As a user, I want the app to identify and tag: document type (innkalling, epikrise, lab-svar, resept, annet), the person the document is for, the issuing institution, and the relevant date. | Must |
| E-4 | As a user, I want to review and edit the extracted text, title, and tags before the document is saved, so I can correct any OCR errors. | Must |
| E-5 | As a user, I want the app to detect which family member the document belongs to based on name/personnummer in the text. | Must |

### Epic 3 – Storage & Sync

| ID | Story | Priority |
|----|-------|----------|
| S-1 | As a user, I want to sign in with my Google account and grant Drive access from within the app. | Must |
| S-2 | As a user, I want to define a root folder in Drive and sub-folders per family member so files go to the right place automatically. | Must |
| S-3 | As a user, I want each document to be stored as both a `.docx` text file and a high-quality image (`.jpeg` or combined `.pdf`) in the appropriate folder. | Must |
| S-4 | As a user, I want uploads to happen in the background so I can continue using the app. | Should |
| S-5 | As a user, I want to see upload status (queued / uploading / done / error) for each document. | Must |
| S-6 | As a user, I want failed uploads to be retried automatically and to receive a notification if they consistently fail. | Should |

### Epic 4 – Library & Search

| ID | Story | Priority |
|----|-------|----------|
| L-1 | As a user, I want to see a chronological list of all documents I have filed, with title, date, type badge, and person. | Must |
| L-2 | As a user, I want to full-text search across all documents so I can find, e.g., all documents mentioning "ortopedi". | Must |
| L-3 | As a user, I want to filter the library by person, document type, and date range. | Must |
| L-4 | As a user, I want to tap a document and read the full extracted text and view the original image. | Must |
| L-5 | As a user, I want to copy the full text of a document to the clipboard so I can paste it into ChatGPT or another AI tool for explanation. | Must |

### Epic 5 – Family & Settings

| ID | Story | Priority |
|----|-------|----------|
| F-1 | As a user, I want to add family member profiles (name, date of birth, optional personnummer) so documents can be routed correctly. | Must |
| F-2 | As a user, I want to map each family member profile to a specific Google Drive folder. | Must |
| F-3 | As a user, I want the app to be available in Norwegian (Bokmål) as the primary language. | Must |

---

## 7. Functional Requirements

### 7.1 Image Capture & Enhancement

- Use AVFoundation for camera access with a document-edge detection overlay (similar to Apple's document scanner).
- Apply perspective transform to produce a flat, rectangular crop.
- Post-processing pipeline: adaptive thresholding for text clarity, shadow removal, deskew correction.
- Output resolution: minimum 300 DPI equivalent for A4 (i.e., ~2480 × 3508 px for a full-page shot).
- Allow export as both JPEG (compressed, ≤ 2 MB per page) and lossless PNG; default to JPEG.

### 7.2 OCR & Text Extraction

- Primary OCR engine: **Apple Vision framework** (`VNRecognizeTextRequest`) — on-device, no data leaves the phone at this stage, supports Norwegian.
- Fallback / accuracy boost: optional **Google Cloud Vision API** (user opt-in, requires connectivity) for complex or degraded documents.
- Preserve approximate layout (paragraph breaks, column structure) in extracted text.
- Confidence score per text block; flag low-confidence segments for user review.

### 7.3 Document Classification & Titling

- Use a local LLM prompt (via Apple Intelligence / on-device model) or a configurable cloud LLM API call to:
  - Identify document type from taxonomy (see §7.6).
  - Extract the document date (prioritise explicit date stamps; fall back to postmark or scan date).
  - Extract the issuing institution (hospital, clinic, fastlege).
  - Extract the patient name and match against family member profiles.
  - Generate a descriptive Norwegian title (5–10 words) summarising the document purpose.
- Output formatted title: `YYYY-MM-DD <Generated Title>` e.g., `2025-11-03 Epikrise ortopedisk poliklinikk St. Olav`.
- User can override any field before saving.

### 7.4 Tagging Schema

Each document receives the following metadata:

| Tag | Type | Example |
|-----|------|---------|
| `person` | String (family member ID) | `"Erik"` |
| `docType` | Enum | `innkalling`, `epikrise`, `lab-svar`, `resept`, `henvisning`, `vedtak`, `annet` |
| `institution` | String | `"St. Olavs hospital"` |
| `specialty` | String (if detectable) | `"Ortopedi"` |
| `documentDate` | ISO 8601 date | `"2025-11-03"` |
| `scanDate` | ISO 8601 datetime | `"2026-02-20T14:32:00Z"` |
| `keywords` | [String] | `["kne", "MR", "operasjon"]` |
| `ocrConfidence` | Float 0–1 | `0.94` |
| `driveFileId` | String | Google Drive file ID |
| `driveImageId` | String | Google Drive image file ID |

Tags are stored locally in a SQLite database (via Core Data) and embedded in the `.docx` file's custom properties.

### 7.5 Google Drive Integration

- Authentication: Google Sign-In SDK, OAuth2 scopes: `drive.file` (access only to files created by the app — privacy-preserving).
- Folder structure (user-configured):
  ```
  <Root folder, e.g. "Helsedokumenter">
  ├── Erik/
  │   ├── Dokumenter/          ← .docx text files
  │   └── Bilder/              ← image files
  └── Liam/
      ├── Dokumenter/
      └── Bilder/
  ```
- File naming: `<title>.docx` and `<title>.jpeg` (title is the generated `YYYY-MM-DD ...` string, sanitised for filesystem).
- If a file with the same name exists, append `(2)`, `(3)`, etc.
- Upload queue: persisted locally so uploads survive app restarts.

### 7.6 Document Type Taxonomy

Norwegian medical document types supported in v1:

| ID | Label (NO) | Description |
|----|-----------|-------------|
| `innkalling` | Innkalling / Innkallingsbrev | Appointment summons |
| `epikrise` | Epikrise | Discharge/clinical summary |
| `lab-svar` | Laboratoriesvar | Lab test results |
| `resept` | Resept / Medisinliste | Prescription or medication list |
| `henvisning` | Henvisning | Referral letter |
| `vedtak` | Vedtak / Enkeltvedtak | Administrative decision |
| `annet` | Annet | Other / unclassified |

### 7.7 `.docx` File Structure

Each generated `.docx` contains:

1. **Header:** Document title, date, institution, person name, document type badge.
2. **Body:** Full OCR-extracted text, preserving paragraph structure. Low-confidence passages marked in yellow highlight.
3. **Footer:** Scan date, app version, original filename reference.
4. **Custom Properties:** All tags from §7.4 as key-value pairs (machine-readable).

---

## 8. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| **Performance** | OCR + classification complete within 10 seconds for a standard A4 page on iPhone 15 |
| **Privacy** | All OCR is on-device by default; no document content sent to external services without explicit user opt-in |
| **Security** | Documents stored locally encrypted using iOS Data Protection (`NSFileProtectionComplete`); Google Drive tokens stored in Keychain |
| **Offline** | Capture, OCR, classification, and local storage work fully offline; Drive upload queued for when connectivity returns |
| **Accessibility** | VoiceOver support for all interactive elements; minimum WCAG AA contrast ratios |
| **Localisation** | Norwegian Bokmål primary; English fallback for UI strings |
| **Data retention** | Local database not deleted on app update; export/backup of local database available in Settings |

---

## 9. Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                     iOS App (Swift/SwiftUI)          │
│                                                     │
│  ┌──────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Capture │  │  Processing  │  │   Library UI  │  │
│  │  Module  │  │  Pipeline    │  │   & Search    │  │
│  └────┬─────┘  └──────┬───────┘  └───────┬───────┘  │
│       │               │                  │          │
│  ┌────▼───────────────▼──────────────────▼───────┐  │
│  │              Local Data Layer                  │  │
│  │   Core Data (SQLite) + File System             │  │
│  └──────────────────────┬────────────────────────┘  │
│                         │                           │
│  ┌──────────────────────▼────────────────────────┐  │
│  │              Sync Service                      │  │
│  │   Upload Queue + Google Drive API Client       │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
         │                              │
   ┌─────▼──────┐               ┌───────▼──────┐
   │ Apple      │               │ Google Drive │
   │ Vision OCR │               │ REST API     │
   │ (on-device)│               │              │
   └────────────┘               └──────────────┘
         │
   ┌─────▼──────────────────┐
   │ On-device LLM          │
   │ (Apple Intelligence or │
   │  configurable API)     │
   └────────────────────────┘
```

### Key Components

| Component | Technology |
|-----------|-----------|
| UI framework | SwiftUI |
| Camera & scanning | AVFoundation + VisionKit (`VNDocumentCameraViewController`) |
| OCR | Apple Vision (`VNRecognizeTextRequest`) — Norwegian language |
| Document classification & titling | Apple Intelligence on-device model (iOS 18+ required); optional OpenAI/Anthropic API fallback (user-supplied key) |
| Local storage | Core Data + FileManager |
| Cloud sync | Google Drive REST API v3 via URLSession |
| Auth | Google Sign-In SDK for iOS |
| `.docx` generation | `libxlsxwriter` / custom `docx` XML writer (lightweight, no server dependency) |
| Search | Core Spotlight + in-app full-text search on Core Data |

---

## 10. Screen Flows

### 10.1 Onboarding Flow

```
App Launch
    └─▶ Welcome screen (Arkivator logo, brief value proposition)
            └─▶ Sign in with Google (Drive access)
                    └─▶ Set up family members (name, DOB; at least one required)
                            └─▶ Map each member to a Drive folder (picker or create new)
                                    └─▶ Home / Library screen
```

### 10.2 Capture & File Flow

```
Tap "+" / Camera button
    └─▶ Camera viewfinder (document edge detection overlay)
            └─▶ [Auto-capture or manual shutter]
                    └─▶ Review & crop screen
                            ├─▶ [Retake]
                            └─▶ [Use photo]
                                    └─▶ Processing screen (OCR + classification, ~5–10s)
                                            └─▶ Review & edit screen
                                                 - Title (editable)
                                                 - Person (picker)
                                                 - Document type (picker)
                                                 - Extracted text (scrollable, editable)
                                                 - Tags (editable chips)
                                                    └─▶ [Save & Upload]
                                                            └─▶ Library (new document added, upload badge)
```

### 10.3 Library & Detail Flow

```
Library screen (list)
    └─▶ Tap document
            └─▶ Detail screen
                 - Title, date, type, person
                 - Extracted text (scrollable)
                 - [View original image]
                 - [Copy text to clipboard]
                 - [Open in Drive]
                 - [Edit metadata]
                 - [Delete]
```

---

## 11. Data Privacy & Compliance

- **GDPR relevance:** Health data is special category personal data under GDPR Art. 9. The app processes this data locally on the user's device on behalf of the data subject themselves (household personal use), placing it under the GDPR's "purely personal or household activity" exception (Art. 2(2)(c)). No data is shared with Arkivator or third parties without explicit consent.
- **On-device processing default:** OCR and classification run entirely on-device. No document content touches Arkivator servers.
- **Cloud storage:** Documents are stored in the user's own Google Drive. The app requests only `drive.file` scope, limiting access to files it created.
- **Opt-in cloud OCR/LLM:** If the user enables enhanced OCR or cloud LLM classification, a clear disclosure is shown explaining that document content will be sent to the selected third-party API.
- **No analytics on document content:** Any analytics (crash reporting, usage metrics) must never include document content, extracted text, or metadata.

---

## 12. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| OCR quality poor on handwritten annotations or faded text | Medium | High | Flag low-confidence text; fallback to cloud OCR opt-in; user can manually edit |
| LLM generates incorrect title or document type | Medium | Medium | User review step before upload; easy correction UI |
| Google Drive API rate limits / quota exceeded | Low | Medium | Exponential backoff; queue management; notify user |
| User stores docs for others (e.g. elderly parent) without consent | Low | High | Show privacy notice during onboarding; user takes responsibility |
| Apple changes Vision API / Apple Intelligence availability | Low | High | Abstract OCR behind a protocol; maintain fallback to Tesseract or cloud |
| Norwegian medical terminology not handled by generic LLM | Medium | Medium | Include Norwegian medical term glossary in classification prompt context |

---

## 13. Future Considerations (v2+)

- **Multi-page document support:** stitch multiple photos into a single paginated `.docx` / PDF.
- **iCloud Drive & Dropbox support** as alternative storage backends.
- **In-app AI assistant:** Tap "Ask about this document" to get a plain-language Norwegian explanation of any document via an integrated LLM chat.
- **HelseNorge / FHIR integration:** Cross-reference scanned documents with the user's official digital health record.
- **Shared family vault:** Invite a co-parent or caregiver to access the same Drive structure.
- **Reminder system:** Parse appointment dates from innkallinger and create calendar events / reminders.
- **Android app** (Flutter rewrite or companion app).
- **Web viewer:** Browse and search the archive via a web interface backed by the same Drive storage.
- **Export to PDF portfolio:** Generate a printable health summary for a new doctor visit.

---

## 14. Open Questions

1. Should the app support Norwegian Nynorsk in addition to Bokmål for UI strings?
2. ~~What is the preferred minimum iOS version?~~ **Decided: iOS 18+** — enables Apple Intelligence on-device LLM for document classification on iPhone 15/16 series.
3. ~~Should the app be free, paid upfront, or subscription-based?~~ **Decided: phased approach** — v1.0 ships as a free personal tool (user supplies their own cloud LLM API key if desired); a future App Store release will use a subscription model that bundles cloud LLM quota for non-technical users.
4. Is there a requirement to support physical accessibility needs (e.g., Braille display, large text) beyond standard VoiceOver?
5. Should the personnummer (national ID) field in family profiles be optional/masked for privacy, or is it needed for reliable person matching?
6. Is there an appetite for a "quick scan" mode that skips the review step and uploads immediately with AI-generated metadata (power-user feature)?

---

*End of document*
