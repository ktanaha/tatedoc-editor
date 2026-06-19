# Detailed Design

| Item | Value |
|---|---|
| Project | TateDoc Editor (working title) |
| Version | 0.1 (draft) |
| Date | 2026-06-19 |
| Prior Documents | Requirements v0.1, Basic Design v0.1 |

---

## 1. Project Structure

```
TateDocEditor/
├─ TateDocEditor.xcodeproj
├─ TateDocEditor/
│  ├─ App/
│  │   ├─ TateDocEditorApp.swift        # @main
│  │   └─ AppDelegate.swift             # NSApplicationDelegate
│  ├─ Document/
│  │   ├─ NovelDocument.swift           # NSDocument subclass
│  │   ├─ DocumentMetadata.swift
│  │   ├─ Chapter.swift
│  │   ├─ Section.swift
│  │   └─ DocumentSettings.swift
│  ├─ Editor/
│  │   ├─ VerticalTextView.swift        # NSTextView subclass
│  │   ├─ VerticalTextViewRepresentable.swift  # SwiftUI bridge
│  │   ├─ ManuscriptOverlayView.swift   # Manuscript paper grid
│  │   ├─ TateLayoutManager.swift       # NSLayoutManager subclass (emphasis dots)
│  │   └─ TextStorageController.swift
│  ├─ Views/
│  │   ├─ EditorView.swift              # Main area
│  │   ├─ OutlineView.swift             # Chapter pane
│  │   ├─ InspectorView.swift           # Right pane
│  │   ├─ RubyInputSheet.swift
│  │   ├─ SearchPanel.swift
│  │   └─ PreferencesView.swift
│  ├─ ViewModels/
│  │   ├─ DocumentViewModel.swift
│  │   ├─ EditorViewModel.swift
│  │   └─ OutlineViewModel.swift
│  ├─ Services/
│  │   ├─ AozoraParser.swift            # Aozora notation -> AttributedString
│  │   ├─ AozoraSerializer.swift        # AttributedString -> Aozora notation
│  │   ├─ ExportService.swift           # txt / md / pdf
│  │   ├─ AutoSaveManager.swift
│  │   └─ FileWrapperBuilder.swift      # .taterdoc package builder
│  ├─ Models/
│  │   ├─ RubyAnnotation.swift
│  │   ├─ EmphasisDotAttribute.swift
│  │   └─ PaperFormat.swift
│  ├─ Extensions/
│  │   ├─ NSAttributedString+Ruby.swift
│  │   └─ NSAttributedString+Aozora.swift
│  └─ Resources/
│      ├─ Assets.xcassets
│      ├─ Info.plist
│      ├─ TateDocEditor.entitlements    # Sandbox configuration
│      └─ Localizable.strings
└─ TateDocEditorTests/
   ├─ AozoraParserTests.swift
   ├─ AozoraSerializerTests.swift
   └─ DocumentTests.swift
```

---

## 2. Key Class Design

### 2.1 Document Layer

```swift
final class NovelDocument: NSDocument {
    @Published var metadata: DocumentMetadata
    @Published var chapters: [Chapter]
    @Published var settings: DocumentSettings

    override func fileWrapper(ofType: String) throws -> FileWrapper
    override func read(from fileWrapper: FileWrapper, ofType: String) throws
    override func makeWindowControllers()
}

struct Chapter: Identifiable, Codable {
    let id: UUID
    var title: String
    var sections: [Section]
    var order: Int
}

struct Section: Identifiable {
    let id: UUID
    var title: String
    var content: NSAttributedString    // Not Codable; custom serialization
    var order: Int
}

struct DocumentMetadata: Codable {
    var title: String
    var author: String
    var createdAt: Date
    var updatedAt: Date
    var totalCharCount: Int
}
```

### 2.2 Editor Layer

```swift
final class VerticalTextView: NSTextView {
    override func awakeFromNib()
    func configureForVerticalLayout()
    override func keyDown(with: NSEvent)    // Adjust arrow-key behavior
}

struct VerticalTextViewRepresentable: NSViewRepresentable {
    @Binding var attributedText: NSAttributedString
    let settings: DocumentSettings

    func makeNSView(context: Context) -> NSScrollView
    func updateNSView(_ nsView: NSScrollView, context: Context)
    func makeCoordinator() -> Coordinator

    final class Coordinator: NSObject, NSTextViewDelegate {
        func textDidChange(_ notification: Notification)
    }
}

final class TateLayoutManager: NSLayoutManager {
    // Custom rendering for emphasis dots and tate-chu-yoko
    override func drawGlyphs(forGlyphRange: NSRange, at: NSPoint)
    private func drawEmphasisDots(...)
}

final class ManuscriptOverlayView: NSView {
    var format: PaperFormat = .standard20x20
    override func draw(_ dirtyRect: NSRect)
    // Render the grid via CoreGraphics
}
```

### 2.3 Service Layer

```swift
struct AozoraParser {
    /// Convert Aozora notation text into NSAttributedString
    func parse(_ source: String) -> NSAttributedString

    private func parseRuby(_ text: String) -> [RubyMatch]
    private func parseEmphasis(_ text: String) -> [EmphasisRange]
    private func parseTatechuyoko(_ text: String) -> [TcyRange]
}

struct AozoraSerializer {
    /// Convert NSAttributedString into Aozora notation text
    func serialize(_ attributed: NSAttributedString) -> String
}

final class FileWrapperBuilder {
    func build(from document: NovelDocument) throws -> FileWrapper
    func parse(_ wrapper: FileWrapper) throws -> NovelDocument
}
```

### 2.4 ViewModel Layer

```swift
final class DocumentViewModel: ObservableObject {
    @Published var document: NovelDocument
    @Published var selectedChapterID: UUID?
    @Published var selectedSectionID: UUID?

    var currentSection: Section? { get }
    func addChapter(title: String)
    func deleteChapter(id: UUID)
    func reorderChapters(from: IndexSet, to: Int)
}

final class EditorViewModel: ObservableObject {
    @Published var attributedText: NSAttributedString
    @Published var charCount: Int

    func insertRuby(base: String, ruby: String, at: NSRange)
    func toggleEmphasis(at: NSRange)
    func toggleTatechuyoko(at: NSRange)
}
```

---

## 3. Key Sequences

### 3.1 File Open

```
User -> AppDelegate: pick file
AppDelegate -> NSDocumentController: openDocument
NSDocumentController -> NovelDocument: read(from:ofType:)
NovelDocument -> FileWrapperBuilder: parse(wrapper)
FileWrapperBuilder -> AozoraParser: parse(each section.txt)
AozoraParser -> NovelDocument: return NSAttributedString
NovelDocument -> makeWindowControllers
WindowController -> EditorView: render
```

### 3.2 Ruby Insertion

```
User -> EditorView: select base, press Cmd+R
EditorView -> RubyInputSheet: present sheet
User -> RubyInputSheet: enter ruby and confirm
RubyInputSheet -> EditorViewModel: insertRuby(base, ruby, range)
EditorViewModel -> NSAttributedString: attach CTRubyAnnotation
EditorViewModel -> VerticalTextView: refresh text
EditorViewModel -> DocumentViewModel: notify change (triggers autosave)
```

### 3.3 Save

```
NSDocument (autosave timer) -> NovelDocument: fileWrapper(ofType:)
NovelDocument -> AozoraSerializer: serialize each section
AozoraSerializer -> return string
NovelDocument -> FileWrapperBuilder: build(document)
FileWrapperBuilder -> FileWrapper: assemble package
NSDocument -> write to disk
```

---

## 4. Key Algorithms

### 4.1 Aozora Notation Parsing

Input example: `今日は｜晴天《せいてん》なり。［＃傍点］注目［＃傍点終わり］してください。`

Steps:
1. **Extract ruby**: Match with regex `｜(.+?)《(.+?)》` -> capture base and reading.
2. **Extract emphasis**: Match `［＃傍点］(.+?)［＃傍点終わり］`.
3. **Extract tate-chu-yoko**: Match `［＃縦中横］(.+?)［＃縦中横終わり］`.
4. **Strip markup**: Build the plain base text by removing markup.
5. **Apply attributes**: Map each extracted range to the correct NSRange in the stripped text, then apply attributes to NSMutableAttributedString.

Notes:
- Disallow overlap between ruby and emphasis ranges (parser warning).
- Recompute offsets relative to the post-stripping text.

### 4.2 Ruby Rendering

```swift
let ruby = CTRubyAnnotationCreateWithAttributes(
    .auto, .auto, .before,
    rubyText as CFString,
    [kCTRubyAnnotationSizeFactorAttributeName: 0.5] as CFDictionary
)
attributedString.addAttribute(
    NSAttributedString.Key(kCTRubyAnnotationAttributeName as String),
    value: ruby,
    range: parentRange
)
```

NSTextView renders `CTRubyAnnotation` automatically via CoreText.

### 4.3 Emphasis Dot Rendering

`CTRubyAnnotation` does not directly support emphasis dots, so:

1. Apply a custom `EmphasisDotAttribute` to the target range.
2. Override `TateLayoutManager.drawGlyphs`.
3. Obtain glyph rects with `boundingRect(forGlyphRange:in:)`.
4. In vertical layout, draw `﹅` to the right of each glyph using CoreGraphics.

### 4.4 Manuscript Paper Grid

- Set `textContainer.lineFragmentPadding` to 0.
- Force a monospaced font (e.g., Hiragino Mincho).
- Per-character width = `font.advancement(forGlyph: "あ".utf16.first!).width`.
- ManuscriptOverlayView draws the grid via CGContext, aligned to character and line spacing.
- In vertical layout, vertical lines represent character columns and horizontal lines represent line rows.

### 4.5 Autosave

- Set NSDocument's `autosavesInPlace = true`.
- Flush 30 seconds (configurable) after a user edit.
- Maintain generation backups every 5 minutes inside Application Support (inside the sandbox).

---

## 5. Data Flow

### 5.1 On Typing

```
[user input] -> VerticalTextView (NSTextStorage updates)
             -> Coordinator.textDidChange
             -> EditorViewModel.attributedText updates
             -> DocumentViewModel propagates
             -> NSDocument.isDocumentEdited = true
             -> autosave timer triggered
```

### 5.2 On Section Switch

```
[OutlineView selection] -> OutlineViewModel.selectedSectionID updates
                       -> DocumentViewModel returns currentSection
                       -> EditorViewModel.attributedText swapped
                       -> VerticalTextView re-renders
```

---

## 6. Sandbox / Entitlements

`TateDocEditor.entitlements`:

| Key | Value | Reason |
|---|---|---|
| `com.apple.security.app-sandbox` | YES | Required for Mac App Store |
| `com.apple.security.files.user-selected.read-write` | YES | User-picked files |
| `com.apple.security.files.bookmarks.app-scope` | YES | Recent file bookmarks |
| `com.apple.security.network.client` | NO | No network in v1.0 |

---

## 7. Test Strategy

### 7.1 Unit Tests (XCTest)

| Target | Concern |
|---|---|
| AozoraParser | Ruby / emphasis / tate-chu-yoko, nesting, malformed input |
| AozoraSerializer | Round-trip with parser |
| FileWrapperBuilder | Save/load equivalence |
| Chapter / Section operations | Reorder, delete, add |

### 7.2 UI Tests

| Scenario | Description |
|---|---|
| Create -> save -> reopen | No data loss |
| Ruby insertion | Ruby appears as expected via the panel |
| Large document | Smooth scrolling at 100,000 characters |

### 7.3 Manual QA

- VoiceOver reading.
- Dark mode switching.
- Behavior on different macOS versions.

---

## 8. Development Phases

| Phase | Content | Rough Duration |
|---|---|---|
| Phase 1 | Project skeleton + vertical layout (NSTextView alone) | 1-2 weeks |
| Phase 2 | NSDocument + file format implementation | 1-2 weeks |
| Phase 3 | Ruby / emphasis / tate-chu-yoko | 2-3 weeks |
| Phase 4 | Outline + chapter management | 1-2 weeks |
| Phase 5 | Manuscript paper mode + export | 1-2 weeks |
| Phase 6 | Tests + Mac App Store submission | 2-3 weeks |

---

## 9. Open Questions (TBD)

- [ ] Default font: Hiragino Mincho vs Yu Mincho.
- [ ] PDF export strategy (hand-rolled CoreText vs NSPrintOperation).
- [ ] UI for highlighting search hits in vertical mode.
- [ ] Version management (Time Machine integration vs in-app generations).
