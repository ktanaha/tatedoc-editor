# TateDoc Editor (working title)

A macOS-native editor specialized for Japanese vertical writing (縦書き). Targeted at novelists, doujinshi authors, and essayists.

> **Status**: 🚧 Design phase complete / implementation not yet started

---

## Overview

TateDoc Editor is a dedicated vertical-writing editor for the Mac App Store. It aims to fill gaps that general-purpose editors leave for writers who work primarily in Japanese vertical text.

### Planned Features (v1.0)

- Vertical text display and editing
- Manuscript paper mode (20x20 / 20x10)
- Input and rendering of ruby, emphasis dots, and tate-chu-yoko
- Chapter and section management (outline)
- Native `.taterdoc` package format with Aozora Bunko notation compatibility

---

## Technology Stack

| Layer | Technology |
|---|---|
| UI | SwiftUI |
| Editor core | AppKit (NSTextView) |
| Ruby rendering | CoreText (CTRubyAnnotation) |
| Persistence | NSDocument + FileWrapper |
| Language | Swift 5.9+ |
| Target OS | macOS 14 (Sonoma) or later (planned) |
| Distribution | Mac App Store |

Architecture: **MVVM + Document-Based Architecture**.

---

## Documentation

Development follows the waterfall model. The pre-implementation design documents are:

| # | Document | Contents |
|---|---|---|
| 01 | [Requirements Specification](docs/design/01_requirements.md) | Goals, target users, functional and non-functional requirements |
| 02 | [Basic Design](docs/design/02_basic_design.md) | System architecture, screens, data model, file format |
| 03 | [Detailed Design](docs/design/03_detailed_design.md) | Class design, key algorithms, test strategy |

---

## Development Phases

| Phase | Content | Status |
|---|---|---|
| 0 | Requirements / Basic Design / Detailed Design | ✅ Complete |
| 1 | Project skeleton + vertical text view (NSTextView only) | ⬜ Not started |
| 2 | NSDocument + file format implementation | ⬜ |
| 3 | Ruby / emphasis / tate-chu-yoko | ⬜ |
| 4 | Outline + chapter management | ⬜ |
| 5 | Manuscript paper mode + export | ⬜ |
| 6 | Testing + Mac App Store submission | ⬜ |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

When filing issues or pull requests, please use the templates under `.github/`.

---

## License

See [LICENSE](LICENSE). Currently All Rights Reserved as a provisional setting that anticipates commercial distribution. OSS licensing will be considered separately.
