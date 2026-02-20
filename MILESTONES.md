# Arkivator v1.0 — Milestones & Tasks

**Version:** 1.0
**Status:** Planning
**Last updated:** 2026-02-20

Each milestone is independently functional and testable. Later milestones build on earlier ones; completing a milestone produces an app that is demonstrably more capable than the one before it.

---

## Milestone 1 — Project Foundation

**Goal:** A runnable iOS app shell with the full data model, navigation skeleton, and developer tooling in place. Every subsequent milestone builds on this foundation without revisiting project setup.

**Testable outcome:** App compiles, launches on iOS 18 simulator, shows a tab bar with placeholder screens, and passes a basic smoke test in CI.

### Tasks

1. Create Xcode project: SwiftUI app target, iOS 18.0 minimum deployment, bundle ID `com.arkivator.app`, Norwegian (Bokmål) as primary localisation.
2. Add Swift Package Manager dependencies: Google Sign-In SDK for iOS, GoogleAPIClientForREST/Drive.
3. Configure SwiftLint with a custom rule scaffold: flag hardcoded strings that look like personnummer patterns (enforcement added properly in Milestone 6).
4. Define Core Data schema:
   - `Document` entity with all fields from PRD §7.4: `id`, `title`, `ocrText`, `ocrConfidence`, `docType`, `person`, `institution`, `specialty`, `documentDate`, `scanDate`, `keywords`, `driveFileId`, `driveImageId`, `localImagePath`, `localDocxPath`, `uploadStatus`.
   - `FamilyMember` entity: `id`, `name`, `relationship`, `driveFolderPath`.
   - `UploadQueueItem` entity: `documentId`, `fileType`, `localPath`, `retryCount`, `lastAttempt`.
5. Scaffold app navigation: `TabView` with three tabs — Library, Capture, Settings — each showing a labelled placeholder view.
6. Add `Info.plist` permission entries: `NSCameraUsageDescription`, `NSPhotoLibraryUsageDescription`, `NSFaceIDUsageDescription`.
7. Create `Localizable.strings` (Bokmål) with a `NSLocalizedString` wrapper utility and add the first batch of UI strings for the tab labels and placeholder screens.
8. Set up XCTest unit-test target; add a smoke test that instantiates the Core Data stack and verifies the schema loads without error.
9. Add `.gitignore` entries for Xcode derived data, user-specific xcuserstate files, and API-key files.
10. Verify the app builds and the smoke test passes in a clean Xcode Cloud / `xcodebuild test` run.

---

## Milestone 2 — Document Capture & Image Enhancement

**Goal:** A user can photograph a paper document with the camera, see it automatically cropped and enhanced, and have the processed image saved to the device.

**Testable outcome:** Launching the Capture tab opens a camera; after shooting, the user sees an enhanced, perspective-corrected JPEG; the file is persisted under `NSFileProtectionComplete`; unit tests verify the image stays within resolution and file-size bounds.

### Tasks

1. Integrate `VNDocumentCameraViewController` (VisionKit) as the primary guided-capture path; present it from the Capture tab.
2. Implement a custom `AVCaptureSession` view with real-time document-edge overlay (Vision `VNDetectRectanglesRequest`) as the advanced capture path.
3. Build image-processing pipeline (Core Image + Vision):
   - Perspective correction and deskew from detected quadrilateral corners.
   - Adaptive thresholding for text contrast.
   - Shadow removal and brightness/contrast normalisation.
4. Enforce minimum output resolution of 2480 × 3508 px (300 DPI equivalent for A4).
5. JPEG export with quality step-down loop to stay at or below 2 MB.
6. Build a `ManualCropView` (SwiftUI overlay with four draggable corner handles) that the user can invoke if the auto-crop is wrong; re-runs the processing pipeline on confirm.
7. Save the enhanced JPEG to `FileManager` under the app's Documents directory with `NSFileProtectionComplete`; store the local path in the corresponding `Document` record (created as a draft at this stage).
8. Show the enhanced image in a minimal post-capture review screen with "Looks good" and "Retake" buttons.
9. Unit tests: verify pipeline output meets resolution lower-bound and file-size upper-bound using a set of fixture images.
10. UI test: tap Capture tab → camera presented → simulate image selection → enhanced image shown.

---

## Milestone 3 — OCR & Text Extraction

**Goal:** After capture, the app extracts Norwegian-language text from the enhanced image, shows it to the user with low-confidence passages highlighted, and lets the user correct it before proceeding.

**Testable outcome:** A captured image produces readable Norwegian text on screen within 10 seconds; low-confidence blocks appear in yellow; manual edits are saved; unit tests validate accuracy on fixture documents.

### Tasks

1. Implement `VNRecognizeTextRequest` with `recognitionLanguages: ["nb-NO", "no", "en-US"]` and `recognitionLevel: .accurate`.
2. Collect `VNRecognizedTextObservation` results; sort by bounding-box vertical then horizontal position to reconstruct reading order.
3. Compute per-block confidence; mark blocks with confidence below 0.70 as low-confidence.
4. Build an `OCRResultView`: a scrollable, editable `TextEditor` with low-confidence spans highlighted in yellow (using `AttributedString`).
5. Display overall OCR confidence percentage in a subtitle below the text editor.
6. Add a "Manual entry" mode toggle: hides the OCR result and shows a blank text field when the user prefers to type the text themselves.
7. Persist `ocrText` (user-edited final value) and `ocrConfidence` (average across blocks) to the draft `Document` record in Core Data.
8. Show an activity indicator and elapsed-time counter during OCR processing.
9. Unit tests: run OCR pipeline against a small fixture corpus of Norwegian medical document images; assert average confidence ≥ 0.90 and character-error rate ≤ 5 %.
10. Integration test: capture fixture image → OCR text appears in `OCRResultView` within 10 seconds.

---

## Milestone 4 — AI Classification & Review Screen

**Goal:** The app automatically infers document type, title, institution, patient name, and document date from the OCR text; the user sees a structured review screen where every field is editable before the document is saved locally.

**Testable outcome:** After OCR, the review screen is pre-populated with a correctly formatted title and classified metadata; the user can edit any field; tapping Save persists the complete `Document` record to Core Data; unit tests cover title formatting and type-mapping rules.

### Tasks

1. Define `DocumentClassifier` protocol: `func classify(ocrText: String) async throws -> DocumentMetadata` where `DocumentMetadata` carries `suggestedTitle`, `docType`, `institution`, `patientName`, `documentDate`.
2. Implement `AppleIntelligenceClassifier` using iOS 18 Foundation Models / Writing Tools integration; include a medical-vocabulary system prompt in Norwegian.
3. Implement `CloudLLMClassifier` fallback: reads a user-supplied API key from Keychain; sends OCR text to the configured endpoint; parses JSON response matching the `DocumentMetadata` schema.
4. Build `ClassificationCoordinator`: tries `AppleIntelligenceClassifier` first; falls back to `CloudLLMClassifier` if unavailable or if the result confidence is below threshold; falls back to blank fields otherwise.
5. Generate document title in the format `YYYY-MM-DD <Generated Title>` (strip forbidden filesystem characters).
6. Map classifier output to one of the seven `DocType` enum values (`innkalling`, `epikrise`, `lab-svar`, `resept`, `henvisning`, `vedtak`, `annet`); default to `annet` when unrecognised.
7. Build `ReviewEditView` (SwiftUI `Form`):
   - Editable title field (pre-filled from classifier).
   - Person text field (free-text, defaults to "Ukjent person" — full family-member picker added in Milestone 7).
   - `DocType` picker showing Norwegian labels.
   - Institution and document-date fields.
   - Full OCR text editor (imported from Milestone 3's `OCRResultView`).
   - Keyword tags token field.
8. "Save & Upload later" CTA: creates final `Document` record in Core Data with `uploadStatus = .queued`; dismisses to Library.
9. "Save offline only" CTA: creates record with `uploadStatus = .localOnly`.
10. Handle classification errors: show inline error banner; allow the user to proceed with all fields blank.
11. Unit tests: title format validation (regex), `DocType` mapping completeness, `ClassificationCoordinator` fallback logic.
12. Integration test: fixture OCR text → `ReviewEditView` pre-populated → save → document appears in Core Data fetch.

---

## Milestone 5 — Document Library & Search

**Goal:** The Library tab shows all saved documents in a searchable, filterable list; tapping a document opens a detail view with full text and image; documents can be permanently and securely deleted.

**Testable outcome:** After saving several documents, the library lists them chronologically; search and filters narrow results correctly; deleting a document removes both the Core Data record and the local files; UI tests cover the full list-detail-delete flow.

### Tasks

1. Build `DocumentListView`: `List` backed by `NSFetchedResultsController`; each row shows title, date, `DocType` badge, person name, and upload-status icon.
2. Default sort: descending by `scanDate`; rows update reactively when new documents are saved.
3. Add `SearchBar` bound to an `NSPredicate` filtering `title` and `ocrText` (case-insensitive, diacritic-insensitive).
4. Build filter toolbar: chips for person (from `FamilyMember` list, plus "All"), `DocType`, and date-range picker; filters compose with search predicate via AND.
5. Index `Document.ocrText` and `Document.title` in Core Spotlight (`CSSearchableItem`) on save; update/delete index entries on document mutation/deletion.
6. Build `DocumentDetailView`:
   - Header: title, date, `DocType` badge, person, institution.
   - Image thumbnail with tap-to-zoom (`fullScreenCover`).
   - OCR text with confidence highlights (reuse `OCRResultView` in read-only mode).
   - Metadata section: scanDate, keywords, upload status, Drive links (greyed out until Milestone 8).
   - Action bar: Copy text (pasteboard), Edit (returns to `ReviewEditView`), Delete.
7. Implement secure delete:
   - Overwrite local JPEG and .docx files with zeroed bytes before `FileManager.removeItem`.
   - Delete Core Data record.
   - Remove Core Spotlight index entry.
   - Show undo snackbar (5-second window); if undone within window, restore record and files from a temporary backup copy.
8. Empty-state view for the library (no documents yet) and for search-no-results.
9. Unit tests: predicate logic for search and each filter type; secure-delete file-overwrite verification.
10. UI test: save two documents → library shows both → search by title returns one → delete one → library shows one → undo restores it.

---

## Milestone 6 — Security & Personnummer Handling

**Goal:** The app is locked behind biometrics/passcode; detected personnummer values are encrypted at rest, masked in the UI, and can be redacted with one tap before export; PII never leaks into logs or filenames.

**Testable outcome:** Backgrounding and re-opening the app triggers biometric authentication; a document containing a personnummer shows the masked form in detail view; one-tap redaction replaces it with `[REDAKTET]` in the local text and regenerates the .docx; unit tests cover regex detection, encryption round-trip, and redaction output.

### Tasks

1. Implement app lock using `LocalAuthentication`: require Face ID / Touch ID / passcode when the app returns to foreground after the configured timeout.
2. Add lock-timeout setting in Settings (`UserDefaults`): "Immediately", "After 1 minute", "After 5 minutes".
3. Build personnummer detection: compile regex for both `DDMMYYNNNNN` (no space) and `DDMMYY NNNNN` (space-separated) patterns; run against `ocrText` during the OCR pipeline (Milestone 3) and surface result to the Review screen.
4. Show a dismissible warning banner in `ReviewEditView` when a personnummer is detected: "Vi fant et personnummer i dokumentet. Vurder å redigere det bort."
5. Generate an AES-256 encryption key on first launch; store it in Keychain under `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`.
6. Encrypt any extracted personnummer value before writing to Core Data; add an `encryptedPersonnummer` attribute to the `Document` entity.
7. Display personnummer in `DocumentDetailView` as `DDMMYY •••••`; provide a "Vis" button that requires Face ID authentication before revealing the full number.
8. Implement one-tap redaction in `DocumentDetailView`: replaces all personnummer occurrences in `ocrText` with `[REDAKTET]`; updates Core Data; marks the local .docx as stale (triggers re-generation in Milestone 8 when Drive sync is available).
9. Ensure personnummer never appears in filenames, `Document.title`, `keywords`, analytics events, or crash reports; add a `PIISanitizer` utility that strips personnummer patterns from any string before it is logged.
10. Integrate `PIISanitizer` into the crash-reporting setup (configured in Milestone 9) so that no personnummer can appear in crash stacks or breadcrumbs.
11. Unit tests: regex detection (valid and invalid inputs), AES-256 encryption/decryption round-trip, redaction leaves no personnummer in output string, `PIISanitizer` strips all variants.

---

## Milestone 7 — Family Members & Settings

**Goal:** The user can add and manage family member profiles; the person picker in the capture review flow is backed by real family-member data; Settings offers Google account management, app-lock configuration, analytics opt-out, and cloud OCR/LLM opt-in with required disclosure.

**Testable outcome:** A family member added in Settings appears in the person picker on the Review screen; deleting a family member retains their historical documents but marks them "Ukjent person"; the full onboarding flow guides a new user from welcome screen to library; UI tests cover add-member, pick-member, and onboarding navigation.

### Tasks

1. Build `FamilyMemberListView` and `FamilyMemberEditView`: add, edit, delete; fields: name (required), relationship (free text), Drive subfolder name (auto-generated from name, editable).
2. Implement `FamilyMember` Core Data CRUD with validation (unique names); deleting a member sets `Document.person` to "Ukjent person" for their documents (no cascade delete).
3. Replace the free-text person field in `ReviewEditView` (Milestone 4) with a `FamilyMember` picker; "Ukjent person" remains a selectable option.
4. Build the complete onboarding flow (`OnboardingCoordinator` using `NavigationStack`):
   1. Welcome screen — app value proposition, "Kom i gang" CTA.
   2. Privacy notice — on-device processing, GDPR household exception, Drive scope explanation.
   3. Google Sign-In step (wires to OAuth2 from Milestone 8; shows a placeholder "Koble til Google Drive" button at this milestone that completes silently).
   4. Add first family member (reuses `FamilyMemberEditView`).
   5. App lock setup — offer Face ID / Touch ID or "Hopp over".
   6. Completion screen → transitions to Library tab.
5. Mark onboarding complete in `UserDefaults`; skip on subsequent launches.
6. Build `SettingsView` with sections:
   - Family members (link to `FamilyMemberListView`).
   - Google account: signed-in account display, Sign Out button (placeholder until Milestone 8).
   - App lock: timeout picker (from Milestone 6).
   - Privacy: analytics opt-out toggle, cloud OCR/LLM opt-in toggle with inline disclosure text.
   - About: app version, open-source licences.
7. Wire analytics opt-out toggle to a `AnalyticsService.shared.isEnabled` flag (SDK integrated in Milestone 9).
8. Wire cloud OCR/LLM opt-in: when enabled, show a one-time consent alert before first use; store consent flag in `UserDefaults`.
9. Haptic feedback (`UIImpactFeedbackGenerator`) on save, delete-confirm, and upload-complete events.
10. UI test: add family member → picker in Review screen shows new member; fresh-install onboarding flow navigates through all steps and lands on Library.

---

## Milestone 8 — Google Drive Sync

**Goal:** Documents saved to the local library are automatically uploaded to the user's Google Drive in a structured folder hierarchy; each document is represented by one .docx and one JPEG; uploads persist across app restarts and retry on failure.

**Testable outcome:** Signing in with Google creates the Drive folder structure; saving a document triggers background upload; the document card shows upload progress and then a "Synced" badge; `driveFileId` and `driveImageId` are persisted; integration tests (against a Drive sandbox or mock) verify the full upload path.

### Tasks

1. Implement Google Sign-In OAuth2 PKCE flow using the Google Sign-In SDK; store the refresh token in Keychain (`kSecAttrAccessibleWhenUnlockedThisDeviceOnly`); wire the onboarding step (Milestone 7, task 4.3) and the Settings Sign Out action.
2. Implement silent token refresh: on each Drive API call, check token expiry; refresh silently if expired; present re-authentication flow only if refresh fails.
3. Build `DriveFolderService`: on first sign-in (and on adding a new family member), create `Arkivator/<MemberName>/Dokumenter/` and `Arkivator/<MemberName>/Bilder/` folders in Drive; cache folder IDs in Core Data / `UserDefaults`.
4. Build `.docx` XML generator (`DocxBuilder`):
   - Header: bold title, document date, institution, person name, `DocType` badge (coloured paragraph border).
   - Body: OCR text; low-confidence passages wrapped in yellow highlight (using `w:rPr` shading).
   - Footer: "Skannet av Arkivator vX.Y — YYYY-MM-DD".
   - Custom document properties: all `Document` tag fields as key-value pairs (§7.4 schema).
   - Write output to `FileManager` under `NSFileProtectionComplete`; store path in `Document.localDocxPath`.
5. Sanitise file names: strip `/ \ : * ? " < > |`; truncate to 240 chars; append ` (2)`, ` (3)` etc. if a file with the same name already exists in the Drive folder.
6. Build `UploadQueue`:
   - Persistent queue backed by the `UploadQueueItem` Core Data entity (survives app termination).
   - Process items sequentially per family member to avoid race conditions on folder creation.
   - Use `URLSession` background upload tasks for JPEG and .docx.
7. Implement retry logic: exponential backoff (2 s, 4 s, 8 s, 16 s) on HTTP 429 / 503 / network errors; mark item as permanently failed after 5 attempts; send a local notification ("Upload mislykkedes — trykk for å prøve igjen").
8. Update upload-status on `Document` record: `.queued` → `.uploading` → `.synced` (with `driveFileId`, `driveImageId` stored) or `.failed`.
9. Show upload status in `DocumentListView` row (queued/uploading spinner, synced cloud icon, failed exclamation badge) and in `DocumentDetailView`.
10. Add "Open in Drive" action in `DocumentDetailView` (enabled only when `driveFileId` is set); opens the Drive URL via `UIApplication.open`.
11. Handle quota-exceeded error: show user-facing alert with link to Drive storage management.
12. Re-generate and re-upload .docx when a document is edited or a personnummer is redacted (sets `uploadStatus` back to `.queued`).
13. Integration test (Drive sandbox / URLProtocol mock): document saved → upload queue item created → upload succeeds → `driveFileId` persisted → "Open in Drive" link is valid.

---

## Milestone 9 — Accessibility, Polish & TestFlight Readiness

**Goal:** The app meets WCAG AA accessibility standards, runs correctly in dark mode, is fully localised in Norwegian Bokmål, and has a comprehensive automated test suite. The milestone concludes with a successful TestFlight submission.

**Testable outcome:** VoiceOver navigation covers all interactive elements without gaps; Dynamic Type does not truncate or overlap text at the largest sizes; all colour combinations pass WCAG AA contrast check; CI runs lint + unit tests + UI tests and archives to TestFlight on every merge to the release branch.

### Tasks

1. Audit all screens with VoiceOver: add `accessibilityLabel`, `accessibilityHint`, and `accessibilityValue` where missing; group related elements with `accessibilityElement(children: .combine)`; ensure logical focus order.
2. Test all text views at Dynamic Type "Accessibility Extra Extra Extra Large"; fix any truncation, overlap, or layout break by switching fixed-height rows to self-sizing layouts.
3. Audit all custom colours with a contrast-ratio tool; ensure text/background pairs meet WCAG AA (4.5:1 for normal text, 3:1 for large text) in both light and dark mode.
4. Implement dark-mode colour assets for every custom `Color`/`UIColor` used in the app; test all screens in dark mode on multiple device sizes.
5. Support `UIAccessibility.isReduceMotionEnabled`: replace animated transitions with cross-dissolves; disable parallax and spring animations when enabled.
6. Conduct performance profiling (Instruments → Time Profiler + Allocations) on iPhone 15 simulator for the OCR + classification pipeline; optimise until the end-to-end processing time is ≤ 10 seconds for a single A4 document.
7. Full Norwegian Bokmål localisation pass: search all source files for un-localised string literals; move all remaining hardcoded UI strings to `Localizable.strings`; get native-speaker review of medical vocabulary.
8. Integrate crash/analytics SDK (Firebase Crashlytics or TelemetryDeck):
   - Configure with app ID; enable on first launch unless analytics opt-out is set.
   - Route all crash reports through `PIISanitizer` (Milestone 6) before upload.
   - Log permitted analytics events: `app_launched`, `document_captured`, `ocr_completed`, `upload_succeeded`, `upload_failed`, feature toggles.
   - Wire the Settings analytics opt-out toggle.
9. Expand XCTest unit-test suite to cover: image pipeline bounds, OCR confidence scoring, `DocumentClassifier` fallback logic, Core Data CRUD, upload-queue persistence and retry counter, AES-256 round-trip, personnummer detection and redaction, `PIISanitizer`, Drive file-name sanitisation and deduplication, `.docx` XML well-formedness.
10. Write XCUITest end-to-end suite for primary user flows:
    - Onboarding: fresh install → sign in → add family member → library visible.
    - Capture → Review → Save → document appears in Library.
    - Search: saved document found by title and by OCR text keyword.
    - Document detail: copy text, edit title, delete with undo.
    - Personnummer: detected → warning shown → one-tap redact → masked in detail view.
11. Set up Xcode Cloud workflow (or Fastlane lanes):
    - On every pull request: `SwiftLint` → unit tests → UI tests (on iPhone 15 simulator).
    - On merge to release branch: archive → TestFlight internal group upload.
12. Prepare App Store Connect configuration:
    - App name: Arkivator, subtitle: "Helsedokumenter, enkelt arkivert".
    - Privacy nutrition labels per §11 of the PRD.
    - Screenshots in Norwegian (6.7" iPhone 15 Pro Max and 6.1" iPhone 15) covering Library, Capture, Review, and Detail screens.
    - App Review notes: on-device OCR/classification, GDPR household-activity exception, `drive.file` scope rationale, demo account credentials.
13. Distribute build to TestFlight internal test group; gather and address blocker-level feedback before declaring v1.0 complete.

---

## Summary

| # | Milestone | Primary PRD Sections | Key Deliverable |
|---|-----------|----------------------|-----------------|
| 1 | Project Foundation | §9, §14 | Runnable app shell, Core Data schema, CI |
| 2 | Document Capture & Enhancement | §7.1, §6 (Epic 1) | Enhanced JPEG saved to device |
| 3 | OCR & Text Extraction | §7.2, §8 | Editable Norwegian text from image |
| 4 | AI Classification & Review Screen | §7.3, §7.4, §6 (Epic 2) | Structured metadata, complete review flow |
| 5 | Document Library & Search | §6 (Epic 4), §8 | Searchable local archive |
| 6 | Security & Personnummer Handling | §8, §11, §15 | Encrypted storage, biometric lock, redaction |
| 7 | Family Members & Settings | §6 (Epic 5), §10 | Multi-person profiles, full onboarding |
| 8 | Google Drive Sync | §7.5, §7.7, §6 (Epic 3) | Cloud backup, .docx export, sync status |
| 9 | Accessibility, Polish & TestFlight | §8, §10, §14 | WCAG AA, full test suite, TestFlight build |
