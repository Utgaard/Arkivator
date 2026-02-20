# Arkivator – Product Requirements Document

**Version:** 1.1
**Date:** 2026-02-20
**Status:** Draft
**Changelog:** v1.1 — added security architecture (§15), App Store publishing path (§14), expanded privacy/compliance (§11), usability patterns (§10.4–10.5), and additional risks/open questions.

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
| **Security – Data at rest** | Documents stored locally encrypted using iOS Data Protection (`NSFileProtectionComplete`); Google Drive tokens and any API keys stored in Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` |
| **Security – App lock** | App requires Face ID / Touch ID or device passcode to unlock after backgrounding or cold launch; configurable auto-lock timeout (immediate / 1 min / 5 min) |
| **Security – Transport** | All network traffic uses TLS 1.2+ with certificate transparency; Google Drive API calls use OAuth2 bearer tokens — never embed client secrets in the binary |
| **Security – Personnummer** | Personnummer (national ID) is stored encrypted in the local database and never included in filenames, Drive file metadata, or analytics payloads; displayed masked in the UI (`DDMMYY •••••`) unless the user explicitly reveals it |
| **Security – Secure deletion** | When a user deletes a document, local files are overwritten before removal; corresponding Drive files are moved to Trash via API (user can permanently delete from Drive) |
| **Offline** | Capture, OCR, classification, and local storage work fully offline; Drive upload queued for when connectivity returns |
| **Accessibility** | VoiceOver support for all interactive elements; Dynamic Type for all text; minimum WCAG AA contrast ratios; support for Bold Text and Reduce Motion system preferences |
| **Localisation** | Norwegian Bokmål primary; English fallback for UI strings; all date/number formatting respects the device locale |
| **Data retention** | Local database not deleted on app update; export/backup of local database available in Settings; iCloud Keychain backup of encryption keys is opt-in only |
| **Reliability** | Crash-free session rate target ≥ 99.5%; upload queue survives app termination and device restart |

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
    └─▶ Welcome screen (Arkivator logo, brief value proposition, 3 swipeable benefit cards)
            └─▶ Privacy notice ("Your documents stay on your device by default")
                    └─▶ Sign in with Google (Drive access) — skip option available*
                            └─▶ Set up family members (name, DOB, optional personnummer)
                                │   At least one member required; "Add myself" quick action
                                └─▶ Map each member to a Drive folder (picker or create new)
                                        └─▶ Set app lock preference (Face ID / passcode)
                                                └─▶ Home / Library screen (with contextual first-scan prompt)
```

\* *If the user skips Google Sign-In, the app functions in offline-only mode. Documents are captured, processed, and stored locally. Drive upload can be enabled later from Settings.*

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

### 10.4 Error Recovery & Edge Cases

| Scenario | Behaviour |
|----------|-----------|
| Camera permission denied | Show a clear explanation screen with a one-tap deep link to iOS Settings |
| OCR returns no text | Display the captured image with a banner: "No text could be extracted. You can enter the details manually or retake the photo." |
| Low OCR confidence (< 0.7 overall) | Highlight uncertain passages in yellow; show a banner suggesting "Review highlighted text — some words may be incorrect" |
| Drive upload fails (network) | Queue the upload; show a yellow "Pending upload" badge; retry automatically with exponential backoff (1s, 2s, 4s, 8s, max 5 retries); notify user after persistent failure |
| Drive upload fails (auth expired) | Prompt the user to re-authenticate with Google; do not discard the queued upload |
| Google Drive quota exceeded | Show a clear error with guidance: "Your Google Drive is full. Free up space or upgrade your storage plan." |
| App killed mid-processing | Persist capture state to disk before OCR begins; on next launch, offer to resume processing |
| Family member not matched | Default to "Ukjent person" (unknown person) and prompt the user to assign before upload |

### 10.5 Usability Patterns

- **Haptic feedback:** Light haptic on successful capture, medium haptic on successful upload, warning haptic on errors.
- **Undo:** After deleting a document from the library, show a 5-second "Undo" snackbar before committing the deletion.
- **Dark mode:** Full support for iOS dark mode, with all custom colours defined as adaptive colour assets.
- **Pull-to-refresh:** Library screen supports pull-to-refresh to re-sync Drive upload statuses.
- **Keyboard avoidance:** All text-editing screens properly adjust for the on-screen keyboard.
- **Empty states:** Library, search results, and family member lists show helpful empty-state illustrations with a clear call to action.

---

## 11. Data Privacy & Compliance

### 11.1 GDPR & Legal Basis

- **GDPR relevance:** Health data is special category personal data under GDPR Art. 9. The app processes this data locally on the user's device on behalf of the data subject themselves (household personal use), placing it under the GDPR's "purely personal or household activity" exception (Art. 2(2)(c)). No data is shared with Arkivator or third parties without explicit consent.
- **Data controller:** The end user is the data controller for all document content. Arkivator (the company/developer) does not act as a data processor for health content, because no content is transmitted to Arkivator infrastructure.
- **Basis for non-content data:** For analytics (crash reporting, anonymised usage events), the legal basis is legitimate interest (Art. 6(1)(f)). Users can opt out in Settings.
- **Right to erasure:** Users can delete all local data via an in-app "Delete All Data" function (Settings → Privacy → Erase Local Archive). This removes all Core Data records, cached images, and queued uploads. A confirmation dialog warns that Drive copies remain unless deleted separately.

### 11.2 Data Flow & On-Device Processing

- **On-device processing default:** OCR and classification run entirely on-device. No document content touches Arkivator servers.
- **Cloud storage:** Documents are stored in the user's own Google Drive. The app requests only `drive.file` scope, limiting access to files it created.
- **Opt-in cloud OCR/LLM:** If the user enables enhanced OCR or cloud LLM classification, a clear disclosure is shown explaining that document content will be sent to the selected third-party API. The user must acknowledge this per-session or permanently via a toggle. The specific API provider and data-handling policy are linked in the disclosure.

### 11.3 Personnummer Handling

- The personnummer (Norwegian national ID, 11 digits) is highly sensitive PII. It is used only for automatic family-member matching during OCR text analysis.
- **Storage:** Stored AES-256 encrypted in the local Core Data store; decrypted only in memory during matching.
- **Display:** Masked by default in all UI surfaces (`DDMMYY •••••`); revealable via a tap + Face ID confirmation.
- **Transmission:** Never included in Drive filenames, file metadata, `.docx` custom properties, analytics payloads, or crash reports. If a personnummer appears in OCR-extracted body text, the user is warned and offered a one-tap redaction.
- **Optional:** The personnummer field in family profiles is optional. If omitted, family-member matching falls back to name-only heuristics.

### 11.4 Analytics & Telemetry

- **No analytics on document content:** Any analytics (crash reporting, usage metrics) must never include document content, extracted text, or metadata.
- **Permitted telemetry:** App launch events, feature-usage counters (e.g., "scanned a document", "searched library"), upload success/failure rates, and crash stack traces.
- **Provider:** Firebase Crashlytics or a privacy-first alternative (e.g., TelemetryDeck) — no Google Analytics.
- **Opt-out:** Users can disable all telemetry from Settings → Privacy.

### 11.5 App Store Privacy Nutrition Labels

The following data types must be declared in App Store Connect (see also §14.5):

| Data Type | Collected | Linked to Identity | Used for Tracking |
|-----------|-----------|-------------------|-------------------|
| Health & Fitness (health records) | Yes (on-device only) | No | No |
| Photos (camera capture) | Yes (on-device only) | No | No |
| Identifiers (Google account ID) | Yes | Yes | No |
| Diagnostics (crash logs) | Yes | No | No |
| Usage Data (feature counters) | Yes | No | No |

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
| **App Store rejection** (health data handling, privacy concerns) | Medium | High | Pre-submission review against App Store Review Guidelines §5.1 (Privacy) and §5.1.1 (Health data); prepare detailed App Review Information notes explaining on-device processing and household-use GDPR exception; maintain a compliance checklist (see §14) |
| **Google Sign-In SDK policy changes** | Low | Medium | Abstract authentication behind a protocol; design storage-provider layer so iCloud/Dropbox can be added as alternatives in v2 |
| **Personnummer leaked via logs/crash reports** | Low | Critical | Strip all PII from crash reports; never log personnummer even at debug level; use static analysis (SwiftLint custom rule) to flag personnummer patterns in string interpolation |
| **User loses device with sensitive health data** | Medium | High | `NSFileProtectionComplete` ensures data is unreadable when device is locked; remote wipe via Find My iPhone erases app data; app lock (Face ID / passcode) prevents casual access |
| **User-supplied API key compromised** | Medium | Medium | Store API keys in Keychain, not UserDefaults; warn user not to share screenshots of Settings; mask key display after entry |
| **Subscription/payment integration issues** | Medium | Medium | Use StoreKit 2 with server-side receipt validation; test extensively with sandbox environment; handle edge cases (family sharing, refunds, grace period) per Apple guidelines |

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

## 14. iOS App Store Publishing Path

This section defines the end-to-end process for publishing Arkivator on the Apple App Store, from developer account setup through post-launch maintenance.

### 14.1 Prerequisites

| Item | Detail |
|------|--------|
| **Apple Developer Program** | Enroll in the Apple Developer Program ($99/year). Required for App Store distribution, TestFlight, and access to App Store Connect. |
| **D-U-N-S Number** | If publishing under a company/organisation, obtain a D-U-N-S number for the Organisation enrollment type. If publishing as an individual developer, this is not required. |
| **Xcode & toolchain** | Xcode 16+ (for iOS 18 SDK); Swift 6; SwiftUI lifecycle |
| **Code signing** | Set up automatic signing in Xcode with the enrolled Apple Developer account; create a Distribution provisioning profile |
| **App ID & bundle identifier** | Register bundle ID (e.g., `no.arkivator.app`) in the Apple Developer portal |
| **Google Cloud project** | Register the app's bundle ID in Google Cloud Console for the Google Sign-In OAuth client; configure the `GIDSignIn` redirect URI scheme |

### 14.2 Development & Testing Milestones

```
Phase 1: Internal Development
    └─▶ Local development builds on physical devices (Developer signing)
    └─▶ Unit tests + UI tests in Xcode (XCTest)
    └─▶ Static analysis: SwiftLint, custom PII-leak rules

Phase 2: TestFlight Beta
    └─▶ Archive & upload build to App Store Connect
    └─▶ Internal testing (up to 100 team members, no App Review required)
    └─▶ External testing (up to 10,000 testers, requires Beta App Review)
    └─▶ Iterate on feedback; fix critical bugs
    └─▶ Validate OCR accuracy on diverse Norwegian medical documents
    └─▶ Validate Google Drive sync reliability across network conditions

Phase 3: App Store Submission
    └─▶ Final pre-submission checklist (see §14.4)
    └─▶ Submit for App Review
    └─▶ Respond to reviewer questions/rejections (if any)
    └─▶ Approved → Release

Phase 4: Post-Launch
    └─▶ Monitor crash reports (Xcode Organizer + Crashlytics)
    └─▶ Phased rollout (optional: 7-day phased release)
    └─▶ Respond to App Store ratings/reviews
    └─▶ Submit updates for bug fixes and v1.x features
```

### 14.3 App Store Connect Configuration

| Field | Value |
|-------|-------|
| **App name** | Arkivator |
| **Subtitle** | Helsedokumenter, enkelt arkivert |
| **Primary language** | Norwegian (Bokmål) |
| **Category** | Productivity (primary) · Health & Fitness (secondary) |
| **Age rating** | 4+ (no objectionable content; health data is user's own) |
| **Price** | Free (v1.0); subscription IAP added in v1.x (see §14.7) |
| **Availability** | Norway initially; expand to Nordics and worldwide as localisation is added |
| **Content rights** | Declare no third-party content; all generated files are user-owned |

### 14.4 Pre-Submission Checklist

- [ ] App runs without crashes on all supported devices (iPhone 15, 15 Pro, 16 series)
- [ ] All required iOS permissions have usage descriptions in `Info.plist`:
  - `NSCameraUsageDescription` — "Arkivator uses the camera to photograph medical documents for archiving."
  - `NSFaceIDUsageDescription` — "Arkivator uses Face ID to protect your health documents."
  - `NSPhotoLibraryUsageDescription` — (only if importing from Photo Library is supported)
- [ ] Privacy Policy URL is live and linked in App Store Connect and in-app Settings
- [ ] Terms of Service URL is live (required for apps with subscriptions)
- [ ] App Store screenshots prepared: 6.7" (iPhone 15 Pro Max) and 6.1" (iPhone 15 Pro) in Norwegian
- [ ] App preview video (optional but recommended): 15–30 second walkthrough of scan → process → upload flow
- [ ] App Review Information notes prepared, explaining:
  - Health data processing is on-device only
  - GDPR household exception applicability
  - Google Drive `drive.file` scope (app cannot read user's existing files)
  - Demo account credentials for reviewers (a Google account with a test Drive folder)
- [ ] Privacy Nutrition Labels completed (see §11.5)
- [ ] No private API usage; no deprecated API usage
- [ ] `NSAppTransportSecurity` does not contain blanket exceptions
- [ ] All third-party libraries have compatible licences (MIT, Apache 2.0, etc.)
- [ ] Minimum deployment target set to iOS 18.0

### 14.5 App Review Guidelines — Key Compliance Areas

| Guideline Section | Relevance | How Arkivator Complies |
|-------------------|-----------|------------------------|
| **1.3 Kids Category** | N/A | App is not in the Kids category |
| **2.1 App Completeness** | Must | All features functional; no placeholder UI; TestFlight beta validates completeness |
| **3.1.1 In-App Purchase** | Must (v1.x) | Subscription uses StoreKit 2; no external payment links; restore purchases supported |
| **4.2 Minimum Functionality** | Must | App provides meaningful utility (OCR, classification, cloud sync) beyond a simple document scanner |
| **5.1 Privacy — Data Collection** | Must | Nutrition labels accurate; data collection minimised; on-device processing by default |
| **5.1.1 Health & Fitness Data** | Must | Health data not sent to third parties; not stored on developer servers; not used for advertising; clear user consent flow |
| **5.1.2 Data Use and Sharing** | Must | `drive.file` scope limits access; no data shared with Arkivator backend; opt-in only for cloud LLM |
| **5.1.3 Health Research** | N/A | App does not conduct health research |

### 14.6 Privacy Policy Requirements

A public privacy policy (hosted at e.g. `https://arkivator.no/privacy`) must cover:

- What data the app collects (camera images, OCR text, metadata, Google account ID, optional personnummer)
- How data is processed (on-device by default; optional cloud LLM with explicit consent)
- Where data is stored (locally on device; user's own Google Drive)
- What data the developer (Arkivator) receives (anonymised crash/usage telemetry only)
- User rights under GDPR (access, rectification, erasure, portability, objection)
- Contact information for data-related inquiries
- Third-party services used (Google Sign-In, optional cloud LLM provider, crash reporting service)

### 14.7 Subscription & Monetisation Path

| Phase | Model | Details |
|-------|-------|---------|
| **v1.0** | Free | Core scanning, OCR (on-device), Drive sync included. User supplies own API key for cloud LLM features. No IAP. |
| **v1.x** | Freemium + Subscription | Free tier retains all v1.0 features. "Arkivator Pro" subscription ($X/month or $Y/year) bundles: cloud LLM quota (no API key needed), priority OCR fallback, multi-page scanning. |
| **v2+** | Subscription tiers | Additional tiers for family sharing, expanded storage integrations (iCloud, Dropbox), and in-app AI assistant. |

**StoreKit 2 implementation:**
- Use `Product.SubscriptionInfo` for subscription management
- Support Family Sharing for subscriptions if applicable
- Handle grace period (billing retry) and offer code redemption
- Implement `Transaction.currentEntitlements` for entitlement checks
- Server-side receipt validation via App Store Server API for fraud prevention

### 14.8 Continuous Delivery Pipeline

| Stage | Tool / Service |
|-------|---------------|
| Source control | Git (GitHub / GitLab) |
| CI/CD | Xcode Cloud or Fastlane + GitHub Actions |
| Build signing | Automatic signing via Xcode Cloud; match (Fastlane) for team environments |
| TestFlight upload | `xcodebuild` archive → `altool` / Xcode Cloud auto-upload |
| App Store submission | App Store Connect API or Fastlane `deliver` |
| Version numbering | Semantic versioning (`MAJOR.MINOR.PATCH`); build number auto-incremented by CI |
| Release notes | Maintained in `CHANGELOG.md`; copied to App Store Connect "What's New" per release |

---

## 15. Security Architecture

### 15.1 Threat Model

| Threat | Attack Surface | Likelihood | Controls |
|--------|---------------|-----------|----------|
| Unauthorised physical access to device | Lock screen, stolen device | Medium | iOS Data Protection (`NSFileProtectionComplete`); app-level biometric lock; auto-lock timeout |
| Network eavesdropping (MITM) | Google Drive API calls, cloud LLM calls | Low | TLS 1.2+ enforced; App Transport Security enabled; certificate transparency |
| Credential theft (Google OAuth tokens) | Keychain, memory | Low | Tokens stored in Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`; no tokens in UserDefaults or logs |
| PII leakage via crash reports | Crashlytics / telemetry | Low | Custom crash-report sanitiser strips text matching personnummer pattern (`\d{11}`), patient names, and document content |
| Malicious input via OCR text | Document content injected into LLM prompts, filenames | Low | Sanitise all OCR text before use in filenames (strip path separators, control characters); LLM prompt injection mitigated by structured prompt templates with clear delimiters |
| Backup exfiltration (iTunes/Finder) | Local backup | Low | Mark sensitive files with `isExcludedFromBackup` or rely on `NSFileProtectionComplete` (encrypted in backup if device passcode is set) |

### 15.2 Key Management

| Secret | Storage | Access Control |
|--------|---------|---------------|
| Google OAuth2 refresh token | iOS Keychain | `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` |
| User-supplied cloud LLM API key | iOS Keychain | `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` |
| Personnummer (encrypted) | Core Data (AES-256, key in Keychain) | Decrypted in memory only during family-member matching |
| Local database encryption key | iOS Keychain | Generated on first launch; never leaves device |
| Google OAuth2 client ID | Compiled into app binary via `GoogleService-Info.plist` | Not secret (public client); client secret is NOT embedded — use PKCE flow |

### 15.3 Authentication Flow

```
App Launch
    └─▶ Check Keychain for valid Google OAuth2 refresh token
            ├─▶ [Valid] Silently refresh access token → proceed
            └─▶ [Missing/expired] Present Google Sign-In screen (ASWebAuthenticationSession)
                    └─▶ OAuth2 Authorization Code + PKCE flow
                            └─▶ Exchange code for tokens
                                    └─▶ Store refresh token in Keychain
                                            └─▶ Proceed to Library
```

- **PKCE (Proof Key for Code Exchange):** Required for public OAuth2 clients (mobile apps cannot securely store a client secret). The Google Sign-In SDK handles PKCE automatically.
- **Token refresh:** Access tokens are short-lived (~1 hour). The app silently refreshes using the stored refresh token before each Drive API call.
- **Revocation:** Users can revoke Drive access from Settings → Account → "Disconnect Google Drive". This deletes local tokens and calls Google's token revocation endpoint.

### 15.4 Data Encryption Summary

```
┌─────────────────────────────────────────────┐
│               iOS Device                     │
│                                             │
│  ┌──────────────────────────┐               │
│  │     iOS Keychain          │               │
│  │  • OAuth tokens           │               │
│  │  • LLM API key           │               │
│  │  • DB encryption key     │               │
│  └──────────────────────────┘               │
│                                             │
│  ┌──────────────────────────┐               │
│  │  Core Data (SQLite)       │  ← Protected │
│  │  • Document metadata      │    by iOS    │
│  │  • Tags, search index     │    Data      │
│  │  • Personnummer (AES-256) │  Protection  │
│  └──────────────────────────┘               │
│                                             │
│  ┌──────────────────────────┐               │
│  │  FileManager              │  ← Protected │
│  │  • Captured images        │    by iOS    │
│  │  • Generated .docx files  │    Data      │
│  │  • Upload queue cache     │  Protection  │
│  └──────────────────────────┘               │
│                                             │
└─────────────────────────────────────────────┘
         │
         │  TLS 1.2+ encrypted
         ▼
┌─────────────────────┐
│  Google Drive        │  ← User's own account
│  (drive.file scope)  │    App-created files only
└─────────────────────┘
```

---

## 16. Open Questions

Items 1–6 carried over from the original draft; items 7–10 are new.

1. Should the app support Norwegian Nynorsk in addition to Bokmål for UI strings?
2. ~~What is the preferred minimum iOS version?~~ **Decided: iOS 18+** — enables Apple Intelligence on-device LLM for document classification on iPhone 15/16 series.
3. ~~Should the app be free, paid upfront, or subscription-based?~~ **Decided: phased approach** — v1.0 ships as a free personal tool (user supplies their own cloud LLM API key if desired); a future App Store release will use a subscription model that bundles cloud LLM quota for non-technical users.
4. Is there a requirement to support physical accessibility needs (e.g., Braille display, large text) beyond standard VoiceOver?
5. ~~Should the personnummer (national ID) field in family profiles be optional/masked for privacy, or is it needed for reliable person matching?~~ **Decided: optional & masked** — personnummer is optional in family profiles; when stored, it is encrypted and masked in the UI (see §11.3).
6. Is there an appetite for a "quick scan" mode that skips the review step and uploads immediately with AI-generated metadata (power-user feature)?
7. Should the app integrate with Apple's HealthKit for structured health record data, or remain document-only?
8. For the subscription model (v2), should cloud LLM usage be metered (per-document) or unlimited within the subscription tier?
9. Should the app offer an in-app personnummer redaction tool for users who want to share `.docx` files with third parties (e.g., a new doctor) but want to strip the national ID first?
10. Is TestFlight-only distribution (no public App Store listing) acceptable for v1.0, with a public listing in v1.1 after user validation?

---

*End of document*
