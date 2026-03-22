# Repository Guidelines

## Project Structure & Module Organization
- `articles/`: 記事のMarkdown。ファイル名は `kebab-case.md`。先頭にFrontmatter（`title`, `emoji`, `published`, `tags`）。
- `books/<slug>/`: 本のチャプター（`*.md`）と設定（`config.yaml`）。`chapters:` の順序に実ファイルが存在することを確認。
- `.claude/`・`CLAUDE.md`: 補助ドキュメント。公開対象外。
- `package.json`: `zenn-cli` 管理。`README.md` は参照用。

## Build, Test, and Development Commands
- 依存インストール: `npm i`
- 記事作成: `npx zenn new:article --slug my-article --title "タイトル"`
- 本作成: `npx zenn new:book --slug llm-app-practice`
- ローカル確認: `npx zenn preview`（`http://localhost:8000`）。保存で自動リロード。

## Coding Style & Naming Conventions
- ファイル名: `kebab-case.md`（例: `ai-spec-driven-development.md`）。
- 見出しは `#` から始め階層は飛ばさない。リストのネストは半角スペース2。
- コードブロックは言語指定（例: ```ts, ```bash）。
- Frontmatter: `title` は簡潔、`emoji` は1つ、`tags` は最大5つ、公開前は `published: false`。

## Testing Guidelines
- `npx zenn preview` で以下を確認:
  - 見出し階層、目次の整合、リンク切れ・画像表示。
  - コードブロックのシンタックスと折返し。
- 本: `books/<slug>/config.yaml` の `chapters` に未存在ファイルがないこと。

## Commit & Pull Request Guidelines
- コミットメッセージ: `[add]`, `[update]`, `[fix]`, `[chore]` + 簡潔な日本語要約（例: `[add] 記事の初稿を追加`）。
- PR には目的・変更範囲・確認手順（`npx zenn preview` での確認ポイント）・スクリーンショット/録画を含める。公開前は必ず `published: false` を維持。

## Agent-Specific Instructions
- 既存記事のスラッグ/パスは変更しない。Frontmatter のキーは保持し値のみ更新。
- 無関係な一括置換・整形は行わない。画像追加時は相対パス運用を推奨。
- 公開可否はレビューで決定し、マージ時に `published: true` に切り替える。

