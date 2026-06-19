# Requirements Specification

| Item | Value |
|---|---|
| Project | TateDoc Editor (working title) |
| Version | 0.1 (draft) |
| Date | 2026-06-19 |
| Target Platform | macOS |
| Distribution | Mac App Store |

---

## 1. Introduction

### 1.1 Purpose
Provide a writing environment optimized for Japanese vertical text on macOS. Address the needs of novel and essay writers that general-purpose editors cannot fully satisfy.

### 1.2 Background
- macOS's built-in TextEdit has limited vertical writing support.
- Most existing vertical writing editors are Windows-only; choices on Mac are scarce.
- Demand from novel publishing platforms and doujinshi creation remains steady.

### 1.3 Scope of This Document
This document defines the requirements that serve as input to the basic design phase. Detailed screen specifications and data structures are defined in the Basic Design document.

---

## 2. Target Users

| Persona | Description | Typical Use Case |
|---|---|---|
| Novelist | Hobbyist or professional novel / light novel writer | Long-form writing, chapter management, revision |
| Doujinshi author | Fan-fiction or original doujin creator | Short to mid-length writing, pre-submission formatting |
| Essay / blog writer | Anyone who prefers vertical layout | Short to mid-length vertical text input |

---

## 3. Functional Requirements

### 3.1 Core Features (Must)

| ID | Feature | Description |
|---|---|---|
| F-01 | Vertical text editing | Edit text flowing top-to-bottom, right-to-left |
| F-02 | Manuscript paper mode | Grid display (e.g., 20x20, 20x10) with character count |
| F-03 | Ruby input | Add and edit ruby (furigana) on base characters |
| F-04 | Emphasis dots | Add emphasis dots (e.g., `﹅`) to selected characters |
| F-05 | Chapter management | Hierarchical chapter/section structure with navigation |
| F-06 | File save | Native package format with export to plain text / Markdown |

### 3.2 Standard Features (Should)

| ID | Feature | Description |
|---|---|---|
| F-07 | Tate-chu-yoko | Display half-width characters horizontally within vertical text |
| F-08 | Character count | Live count per chapter and total |
| F-09 | Find & replace | Works in vertical layout |
| F-10 | Outline view | Sidebar tree of chapters and sections |
| F-11 | Autosave | Periodic background save |
| F-12 | Dark mode | Follow macOS appearance setting |

### 3.3 Extended Features (Could)

| ID | Feature | Description |
|---|---|---|
| F-13 | Templates | Format selection on document creation |
| F-14 | Backup generations | Rolling automatic backups |
| F-15 | PDF export | Preserve vertical layout when exporting to PDF |
| F-16 | iCloud sync | Through iCloud Drive (document folder) |

---

## 4. Non-Functional Requirements

### 4.1 Performance
- Startup time within 3 seconds on a typical Mac.
- Smooth scrolling and editing on documents up to 100,000 characters.
- Autosave runs in the background without blocking the UI.

### 4.2 Usability
- Comply with macOS Human Interface Guidelines.
- Follow standard macOS shortcuts (e.g., Cmd+S, Cmd+Z).
- Main editing operations must be possible via keyboard alone.

### 4.3 Reliability
- Recover edits after a crash (recovery mode).
- Guarantee explicit manual save in addition to autosave.

### 4.4 Security
- Comply with App Sandbox.
- File access limited to what the user explicitly grants.
- No network communication in v1.0.

### 4.5 Supported Environment
- macOS 14 (Sonoma) or later (TBD).
- Universal: Apple Silicon and Intel.

### 4.6 Internationalization
- v1.0 ships with Japanese UI only.
- Internal design must allow future localization.

---

## 5. Constraints

- Must comply with Mac App Store Review Guidelines.
- File access must go through Sandbox-allowed APIs.
- Vertical layout rendering depends on the NSTextView family of standard frameworks.

---

## 6. Out of Scope (v1.0)

- Windows / iOS / iPadOS versions.
- Real-time collaborative editing.
- Cloud-based document management (other than iCloud Drive).
- Professional typesetting / printing (PDF export is the substitute).
- AI-assisted writing.

---

## 7. Glossary

| Term | Definition |
|---|---|
| Vertical writing | Layout where characters flow top to bottom and lines proceed right to left |
| Ruby (furigana) | Small reading aid characters attached to base characters |
| Emphasis dots | Dots placed beside characters to indicate emphasis |
| Tate-chu-yoko | A piece of horizontal text embedded inside vertical text |
| Manuscript paper mode | Editing mode that overlays a grid resembling Japanese manuscript paper |

---

## 8. Open Questions (TBD)

To be decided before moving to the next phase:
- [ ] Minimum supported macOS version.
- [ ] Native file format encoding (XML / JSON / custom binary).
- [ ] Pricing model (one-time / subscription / free + IAP).
- [ ] Required level of accessibility support.
