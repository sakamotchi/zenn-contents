# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## リポジトリ概要

このリポジトリは、Zenn.dev (https://zenn.dev) に技術記事や本を公開するためのZennコンテンツリポジトリです。コンテンツはYAMLフロントマター付きのMarkdown形式で記述します。

## 必須コマンド

### コンテンツ管理
```bash
# ブラウザでコンテンツをローカルプレビュー
npx zenn preview

# 新しい記事を作成
npx zenn new:article

# 新しい本を作成
npx zenn new:book

# 記事の一覧を表示
npx zenn list:articles

# 本の一覧を表示
npx zenn list:books
```

## リポジトリ構造

```
.
├── articles/          # 個別の記事（単独の投稿）
│   └── *.md          # YAMLフロントマター付きの記事ファイル
├── books/            # 本のプロジェクト（複数チャプターのコンテンツ）
│   └── [book-slug]/  # 各本は独自のディレクトリを持つ
│       ├── config.yaml       # 本のメタデータとチャプター順序
│       └── [chapter].md      # チャプターのMarkdownファイル
└── package.json      # zenn-cli依存関係を含む
```

## コンテンツフォーマット要件

### 記事 (`articles/*.md`)

記事は以下の構造のYAMLフロントマターが必須です:

```yaml
---
title: "記事のタイトル"
emoji: "🗃️"
type: "tech"  # tech: 技術記事 / idea: アイデア記事
topics: ["AI", "Claude", "SQL"]  # トピックタグの配列
published: true  # または false（下書き）
---
```

### 本 (`books/[slug]/`)

各本のディレクトリには以下が含まれます:

1. **config.yaml** - 本のメタデータ:
```yaml
title: '本のタイトル'
summary: '本の要約'
topics: ['トピック1', 'トピック2']
published: false  # または true（公開）
price: 0  # 無料の場合は0、有料の場合は200〜5000
chapters:
  - chapter-slug-1
  - chapter-slug-2
```

2. **チャプターファイル** - config.yamlのスラグに従って命名（例: `01-preparation.md`）

## コンテンツガイドライン

- 記事は `articles/` 内の単一のMarkdownファイル
- 本は `books/` 下のサブディレクトリに複数のチャプターで構成
- チャプターの順序は `config.yaml` の `chapters` 配列で制御
- メタデータにはYAMLフロントマターを使用し、Markdownコンテンツ内には含めない
- ファイル名がスラグ/URL識別子になる（kebab-caseを使用）

## 開発ワークフロー

1. `npx zenn new:article` または `npx zenn new:book` で新しいコンテンツを作成
2. Markdownファイルを編集してコンテンツを作成
3. `npx zenn preview` でローカルで変更をプレビュー
4. コミットしてプッシュすると公開される（ZennはGitHubと同期）
