# Basic Design

| Item | Value |
|---|---|
| Project | TateDoc Editor (working title) |
| Version | 0.1 (draft) |
| Date | 2026-06-19 |
| Prior Document | Requirements Specification v0.1 |

---

## 1. System Architecture

### 1.1 Technology Stack

| Layer | Technology | Notes |
|---|---|---|
| UI framework | SwiftUI | Primary UI construction |
| Editor core | AppKit (NSTextView) | Required for native vertical text |
| Bridge | NSViewRepresentable | SwiftUI ↔ AppKit |
| Ruby rendering | CoreText (CTRubyAnnotation) | Attached as an attribute on NSAttributedString |
| Persistence | NSDocument + FileWrapper | Document-based app |
| Language | Swift 5.9+ | |
| Minimum OS | macOS 14 (Sonoma) | Pending finalization (TBD-1) |

### 1.2 Architectural Pattern

**MVVM + Document-Based Architecture.**

```
┌─────────────────────────────────────────┐
│  View (SwiftUI)                         │
│  ├─ EditorView                          │
│  ├─ OutlineView                         │
│  └─ InspectorView                       │
└──────────────┬──────────────────────────┘
               │ @ObservedObject
┌──────────────▼──────────────────────────┐
│  ViewModel                              │
│  ├─ DocumentViewModel                   │
│  └─ EditorViewModel                     │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Model (NSDocument)                     │
│  ├─ NovelDocument                       │
│  ├─ Chapter / Section                   │
│  └─ AttributedText (ruby / emphasis)    │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Infrastructure                         │
│  ├─ FileWrapper (persistence)           │
│  ├─ AutoSaveManager                     │
│  └─ ExportService                       │
└─────────────────────────────────────────┘
```

---

## 2. Screen Composition

### 2.1 Main Window Layout

```
┌──────────────────────────────────────────────────────┐
│ [Toolbar: New / Save / Format / Ruby / Emphasis ...]│
├────────┬──────────────────────────────┬──────────────┤
│        │                              │              │
│ Outline│                              │  Inspector   │
│ pane   │                              │  (optional)  │
│        │     Editor area              │              │
│ * Prol │     (vertical text)          │  Char count  │
│ * Ch 1 │                              │  Chapter info│
│   - s1 │                              │  Ruby list   │
│   - s2 │                              │              │
│ * Ch 2 │                              │              │
│        │                              │              │
├────────┴──────────────────────────────┴──────────────┤
│ [Status bar: char count / cursor pos / save state]  │
└──────────────────────────────────────────────────────┘
```

### 2.2 Screen Inventory

| ID | Name | Purpose |
|---|---|---|
| S-01 | Main window | Main editor surface |
| S-02 | New document sheet | Template selection |
| S-03 | Preferences | Fonts, grid, autosave settings |
| S-04 | Ruby input sheet | Sheet to input base and ruby |
| S-05 | Find / replace panel | Slide-in search bar |
| S-06 | Export sheet | Format selection (txt / md / pdf) |

### 2.3 Navigation
- Each main window corresponds to one NSDocument; multiple documents can be open simultaneously.
- Panels are presented modally or as inspectors within the same window.

---

## 3. Functional Specifications

### 3.1 Vertical Text Editing (F-01)
- Set NSTextView's `layoutOrientation` to `.vertical`.
- Scrolling is horizontal.
- Cursor movement: up/down moves within a line; left/right moves between lines (matching the vertical reader's mental model).

### 3.2 Manuscript Paper Mode (F-02)
- Render a grid using a CALayer on top of NSTextView.
- Selectable formats: 20x20, 20x10, or custom.
- Character count is synchronized with cell count.

### 3.3 Ruby Input (F-03)
- Select base characters, then Cmd+R or use the toolbar to open the Ruby panel.
- After input, attach `kCTRubyAnnotationAttributeName` to the corresponding range.
- On persistence, convert to Aozora Bunko notation (e.g., `｜base《ruby》`).

### 3.4 Emphasis Dots (F-04)
- Apply a custom attribute on the NSAttributedString range.
- Render via a subclass of NSLayoutManager.
- Persist using Aozora Bunko notation (e.g., `［＃傍点］...［＃傍点終わり］`) for compatibility.

### 3.5 Chapter Management (F-05)
- Two-level hierarchy: Chapter > Section.
- Outline pane uses SwiftUI `List` + `OutlineGroup`.
- Drag to reorder.
- Each section owns an independent NSAttributedString.

### 3.6 File Save (F-06)
- Native format: **`.taterdoc`** (package format).
- Export: `.txt`, `.md`, `.pdf` (the last as an extended feature).
- Autosave: rely on NSDocument's built-in autosaving.

---

## 4. Data Model

### 4.1 Main Entities

```
NovelDocument (NSDocument)
├─ metadata: DocumentMetadata
│   ├─ title: String
│   ├─ author: String
│   ├─ createdAt: Date
│   └─ updatedAt: Date
├─ chapters: [Chapter]
│   └─ Chapter
│       ├─ id: UUID
│       ├─ title: String
│       ├─ sections: [Section]
│       └─ order: Int
└─ settings: DocumentSettings
    ├─ paperFormat: PaperFormat
    ├─ font: FontInfo
    └─ rubyStyle: RubyStyle

Section
├─ id: UUID
├─ title: String
├─ content: NSAttributedString (body + ruby / emphasis attributes)
└─ order: Int
```

### 4.2 File Format (`.taterdoc`)

Adopt a package format (FileWrapper):

```
novel.taterdoc/
├─ document.json        # metadata + structure
├─ settings.json        # document settings
└─ contents/
    ├─ {section-uuid-1}.txt   # Aozora-Bunko based notation
    ├─ {section-uuid-2}.txt
    └─ ...
```

**Rationale**:
- A single binary file is hard to debug.
- A package keeps section bodies as plain text—readable, Git-friendly.
- Aozora notation ensures export compatibility with the ecosystem.

### 4.3 Aozora Bunko Notation (adopted subset)

| Element | Notation |
|---|---|
| Ruby | `｜base《ruby》` |
| Emphasis dots | `［＃傍点］...［＃傍点終わり］` |
| Tate-chu-yoko | `［＃縦中横］...［＃縦中横終わり］` |
| Chapter break | `［＃改丁］` |

---

## 5. External Interfaces

### 5.1 I/O

| Type | Format | Purpose |
|---|---|---|
| Import | `.txt` (UTF-8), `.taterdoc` | Open existing manuscripts |
| Export | `.txt`, `.md`, `.pdf` | Publication / submission |
| Auto backup | `.taterdoc` (generations) | Stored inside the sandbox |

### 5.2 OS Integration

| Feature | API |
|---|---|
| File open / save | NSOpenPanel / NSSavePanel |
| Autosave | NSDocument autosaving |
| Dark mode | NSAppearance observation |
| Spotlight | CoreSpotlight (deferred for v1.0 consideration) |

---

## 6. Error Handling Policy

| Category | Policy |
|---|---|
| Read failure | Show an alert; offer a recovery mode |
| Save failure | Retry; fall back to a temporary backup location |
| Crash | Recover via NSDocument autosaving |
| Invalid notation | Highlight the offending range during parsing but continue |

---

## 7. Open Questions (TBD)

- [ ] Remember the visibility state of the outline pane?
- [ ] Implementation strategy for PDF export (custom CoreText vs WebKit-based).
- [ ] Whether iCloud Drive sync ships in v1.0.
- [ ] App icon and branding.
