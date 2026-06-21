# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working in the **tatedoc-editor** repository.

> 親ディレクトリ `/Users/tanahashitakashi/work/CLAUDE.md` はワークスペース共通の汎用ガイド（Go/Python/TS 中心）です。本プロジェクトは macOS ネイティブの Swift アプリのため、ビルド・規約はこのファイルを優先してください。

---

## Project Overview

TateDoc Editor は、日本語縦書きに特化した macOS ネイティブエディタ（Mac App Store 配信を想定）。小説家・同人作家・エッセイストが対象。

- **現状**: 設計フェーズ完了 / 実装未着手（Phase 0 → Phase 1 へ移行する段階）
- **開発モデル**: ウォーターフォール（要件 → 基本設計 → 詳細設計 → 実装）
- **アーキテクチャ**: MVVM + Document-Based Architecture

設計の一次情報は必ず `docs/design/` を参照すること。実装判断は設計ドキュメントに従い、逸脱する場合はドキュメントを先に更新する。

| # | Document | Contents |
|---|---|---|
| 01 | `docs/design/01_requirements.md` | 要件定義（目的・対象・機能/非機能要件） |
| 02 | `docs/design/02_basic_design.md` | 基本設計（構成・画面・データモデル・ファイル形式） |
| 03 | `docs/design/03_detailed_design.md` | 詳細設計（クラス設計・主要アルゴリズム・テスト戦略） |

---

## Technology Stack

| Layer | Technology |
|---|---|
| UI | SwiftUI |
| Editor core | AppKit (`NSTextView`) ※縦書きに必須 |
| Bridge | `NSViewRepresentable`（SwiftUI ↔ AppKit） |
| Ruby rendering | CoreText（`CTRubyAnnotation`） |
| Persistence | `NSDocument` + `FileWrapper` |
| Language | Swift 5.9+（ローカルは Swift 6.3 系を確認） |
| Target OS | macOS 14 (Sonoma) 以降（TBD-1: 最低バージョン未確定） |
| Distribution | Mac App Store（App Sandbox 必須） |

---

## Project Structure (planned)

詳細設計 §1 のディレクトリ構成に従う。実装時は以下のレイヤ分けを維持すること。

```
TateDocEditor/
├─ App/          # @main, AppDelegate
├─ Document/     # NSDocument サブクラス, Chapter/Section, メタデータ
├─ Editor/       # VerticalTextView, LayoutManager, 原稿用紙オーバーレイ
├─ Views/        # SwiftUI ビュー（Editor/Outline/Inspector/各 Sheet）
├─ ViewModels/   # DocumentViewModel, EditorViewModel, OutlineViewModel
├─ Services/     # AozoraParser/Serializer, ExportService, AutoSaveManager
├─ Models/       # RubyAnnotation, EmphasisDotAttribute, PaperFormat
├─ Extensions/   # NSAttributedString 拡張
└─ Resources/    # Assets, Info.plist, entitlements, Localizable.strings
TateDocEditorTests/  # AozoraParser/Serializer/Document の XCTest
```

> まだ Xcode プロジェクトは未生成。`docs/design/03_detailed_design.md` §1 が唯一の正。レイヤ責務（View / ViewModel / Model / Service）を越えた依存を作らないこと。

---

## Build and Development Commands

> **重要**: この環境には現在 **Xcode 本体（`xcodebuild`）が未導入**で、Command Line Tools の `swift` のみが利用可能。SwiftUI/AppKit の GUI アプリのビルド・署名・App Store 申請には Xcode 本体が必要。GUI ビルドが必要なコマンドは、Xcode 導入後に下記が有効になる。

```bash
# プロジェクト生成後（Xcode 必須）
xcodebuild -scheme TateDocEditor -configuration Debug build
xcodebuild -scheme TateDocEditor -destination 'platform=macOS' test
open TateDocEditor.xcodeproj      # Xcode で開く

# Swift ツールチェーン単体で可能なこと（ロジック層の検証）
swift --version
# Service/Model などプラットフォーム非依存ロジックは SPM 化すると CLI でテスト可能
```

ローカルで Xcode が使えない場合は、AppKit/SwiftUI に依存しない純ロジック（AozoraParser/Serializer 等）の設計・実装・単体テストを優先し、GUI 結合は Xcode 環境で行う方針とする。

---

## Testing and Quality

詳細設計 §7 のテスト戦略に従う。

```bash
# 単体テスト（XCTest, Xcode 環境）
xcodebuild -scheme TateDocEditor -destination 'platform=macOS' test
```

| 対象 | 検証内容 |
|---|---|
| AozoraParser | ルビ / 傍点 / 縦中横、入れ子、不正入力 |
| AozoraSerializer | Parser との往復（round-trip）一致 |
| FileWrapperBuilder | 保存/読込の同値性 |
| Chapter / Section 操作 | 並び替え・削除・追加 |

- Lint/Format（SwiftLint / SwiftFormat）は **未導入**。導入する場合は `.swiftlint.yml` / `.swiftformat` をリポジトリ直下に追加し、本節に実行コマンドを追記する。
- 青空文庫記法は往復一致を必ずテストで担保すること（記法の取りこぼしはデータ欠損に直結）。

---

## Key Domain Knowledge

実装前に押さえるべきドメイン要点（詳細は設計ドキュメント参照）。

- **縦書き**: `NSTextView.layoutOrientation = .vertical`。スクロールは水平。矢印キー挙動は縦書き読者のメンタルモデルに合わせて調整（詳細設計 §2.2）。
- **ルビ**: `CTRubyAnnotation` を `NSAttributedString` の属性として付与。CoreText が自動描画（詳細設計 §4.2）。
- **傍点**: `CTRubyAnnotation` では表現できないため、カスタム属性 + `NSLayoutManager` サブクラスで描画（詳細設計 §4.3）。
- **青空文庫記法**: 永続化フォーマットの基盤。ルビ `｜base《ruby》`、傍点 `［＃傍点］…［＃傍点終わり］`、縦中横 `［＃縦中横］…［＃縦中横終わり］`、改丁 `［＃改丁］`（基本設計 §4.3）。
- **ファイル形式 `.taterdoc`**: `FileWrapper` によるパッケージ形式。`document.json` + `settings.json` + `contents/{uuid}.txt`（本文は青空記法プレーンテキスト）。単一バイナリにしない理由は基本設計 §4.2。
- **Sandbox / Entitlements**: App Sandbox 必須。ネットワークは v1.0 では無効（`network.client` = NO）。詳細設計 §6。

---

## Development Phases

| Phase | Content | Status |
|---|---|---|
| 0 | 要件 / 基本設計 / 詳細設計 | ✅ Complete |
| 1 | プロジェクト雛形 + 縦書きビュー（NSTextView のみ） | ⬜ Not started |
| 2 | NSDocument + ファイル形式実装 | ⬜ |
| 3 | ルビ / 傍点 / 縦中横 | ⬜ |
| 4 | アウトライン + 章管理 | ⬜ |
| 5 | 原稿用紙モード + エクスポート | ⬜ |
| 6 | テスト + Mac App Store 申請 | ⬜ |

各 Phase の粒度・目安期間は詳細設計 §8 を参照。

---

## Git / Contribution Workflow

`CONTRIBUTING.md` に準拠する。

- **ブランチ戦略**: `main`（常にリリース可能） / `develop`（統合） / `feature/<name>` / `fix/<name>` / `docs/<name>`
  - 実装は `develop` から `feature/...` を切る。`main` へは直接コミットしない。
  - リモート: `origin` = `github.com/ktanaha/tatedoc-editor`
- **コミット規約**: Conventional Commits（`feat` / `fix` / `docs` / `refactor` / `test` / `chore`）
  - 例: `feat(editor): add vertical layout to NSTextView`
- **PR / Issue**: `.github/` 配下のテンプレートを使用。
- **設計変更**: `docs/` を変更する場合は対象ドキュメント冒頭に変更ログを追記する（CONTRIBUTING）。

---

## Pre-Commit Checklist

- [ ] テスト通過（`xcodebuild ... test`、Xcode 環境）
- [ ] レイヤ責務（View/ViewModel/Model/Service）を越えた依存を作っていない
- [ ] 設計から逸脱する場合、先に `docs/design/` を更新した
- [ ] 青空記法に関わる変更は往復一致テストを追加/更新した
- [ ] `main` ではなく `feature/` または `fix/` ブランチで作業している
- [ ] コミットメッセージが Conventional Commits 準拠
- [ ] `.env` 等の秘匿ファイルをコミットしていない

---

## Open Questions (TBD)

実装方針に影響する未確定事項（各設計ドキュメントの "Open Questions" を集約）。

- [ ] 最低 macOS バージョン（要件 §8）
- [ ] ネイティブファイル形式のエンコーディング（XML / JSON / 独自）（要件 §8）
- [ ] 価格モデル（買い切り / サブスク / 無料+IAP）（要件 §8）
- [ ] PDF エクスポート実装方式（自作 CoreText vs WebKit/NSPrintOperation）（基本設計 §7, 詳細設計 §9）
- [ ] デフォルトフォント（ヒラギノ明朝 vs 游明朝）（詳細設計 §9）
- [ ] iCloud Drive 同期を v1.0 に含めるか（基本設計 §7）
- [ ] Xcode プロジェクト管理方式（.xcodeproj 直接 / XcodeGen / SwiftPM 中心）← 開発体制の決定事項

---

## Context Compaction

コンテキスト圧縮時に保持すべき情報:
1. 変更したファイルの一覧とパス
2. 実行したテストコマンドと結果
3. 発生したエラーメッセージ
4. アーキテクチャ上の決定事項（特に設計ドキュメントからの逸脱と理由）
