# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

Zenn.dev (https://zenn.dev) に技術記事や本を公開するためのコンテンツリポジトリ。コンテンツはYAMLフロントマター付きのMarkdown形式で記述する。mainブランチにpushすると自動的にZennに公開される。

## コマンド

```bash
npx zenn preview          # ブラウザでローカルプレビュー（http://localhost:8000）
npx zenn new:article      # 新しい記事を作成
npx zenn new:book         # 新しい本を作成
npx zenn list:articles    # 記事の一覧を表示
npx zenn list:books       # 本の一覧を表示
```

## コンテンツフォーマット

### 記事 (`articles/*.md`)

```yaml
---
title: "記事のタイトル"
emoji: "🗃️"
type: "tech"  # tech: 技術記事 / idea: アイデア記事
topics: ["AI", "Claude", "SQL"]  # トピックタグの配列（最大5つ）
published: true  # false で下書き
---
```

### 本 (`books/[slug]/`)

- `config.yaml`: タイトル、要約、トピック、公開状態、価格（0=無料、200〜5000=有料）、チャプター順序を定義
- チャプターファイル: config.yamlの `chapters` 配列のスラグに対応する `.md` ファイル

## コンテンツガイドライン

- ファイル名がURL識別子（スラグ）になるため、kebab-caseを使用する
- 記事は日本語で執筆する
- メタデータはYAMLフロントマターに記述し、Markdown本文中には含めない
- `published: true` にしてpushすると即座に公開されるため注意する
