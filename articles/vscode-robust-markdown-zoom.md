---
title: "AI Safe Markdown Previewにズームイン・ズームアウト機能を追加した"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["VSCode", "Markdown", "Mermaid", "TypeScript", "AI"]
published: true
publication_name: "third_tech"
---

# はじめに

以前、AIエージェントの高頻度更新に耐えるMarkdownプレビューVSCode拡張「**AI Safe Markdown Preview**」を作った記事を書きました。

https://zenn.dev/third_tech/articles/vscode-robust-markdown-introduction

このたび、v0.4.0でプレビューに**ズームイン・ズームアウト機能**を追加したので紹介します。

https://github.com/sakamotchi/vscode-robust-markdown

# 追加した機能：プレビューのズーム

プレビューの文字サイズを **50%〜300%** の範囲で拡大・縮小できるようになりました。

- 大きなモニターで小さく見える文字を拡大して読みやすくする。
- 全体構成を俯瞰したいときに縮小する。

といった使い方ができます。

## 操作方法

ズームは3つの方法で操作できます。

| 操作 | 方法 |
|------|------|
| ズームイン | プレビュー上部の `＋` ボタン / `Ctrl`（macは`Cmd`）+ ホイール上 |
| ズームアウト | プレビュー上部の `−` ボタン / `Ctrl`（macは`Cmd`）+ ホイール下 |
| 100%にリセット | 中央のパーセント表示（例「100%」）をクリック |

ズームレベルは10%刻みで変化し、現在のズーム率は上部バーに常に表示されます。なお、プレビューパネルを開き直すと毎回100%にリセットされます。

# 実装のポイント：font-sizeスケーリング

ズームの実装方法としては、ビューポート全体を拡大縮小する方法も考えられますが、それだとテーブルやMermaidダイアグラムのレイアウトが崩れやすくなります。

そこでこの拡張では、コンテンツの **`font-size` をCSSカスタムプロパティ（`--zoom`）でスケーリングする**方式を採用しました。

```css
#content { font-size: var(--zoom, 100%); }
```

`font-size` を基準にすることで、テキストの折り返し・テーブル・Mermaidダイアグラムが比率を保ったまま拡大縮小され、レイアウトが崩れません。

## JavaScript側の処理

ズームのロジックはWebview内のJavaScriptで完結しています。VSCodeのコマンドやキーバインドは使わず、Webview内のボタンクリックとホイールイベントだけで動作します。

```javascript
var ZOOM_MIN = 50;
var ZOOM_MAX = 300;
var ZOOM_STEP = 10;
var zoomLevel = 100;

function applyZoom() {
  document.getElementById('content').style.setProperty('--zoom', zoomLevel + '%');
  var label = document.getElementById('zoom-level');
  if (label) { label.textContent = zoomLevel + '%'; }
}

function changeZoom(delta) {
  zoomLevel = Math.min(ZOOM_MAX, Math.max(ZOOM_MIN, zoomLevel + delta));
  applyZoom();
}
```

`changeZoom()` で `Math.min` / `Math.max` を使い、ズームレベルが50%〜300%の範囲を超えないように制御しています。

`Ctrl` / `Cmd` + ホイールについては、`wheel` イベントで修飾キーが押されているときだけ既定の動作を抑制してズームに割り当てています。

```javascript
window.addEventListener('wheel', function(event) {
  if (!event.ctrlKey && !event.metaKey) { return; }
  event.preventDefault();
  changeZoom(event.deltaY < 0 ? ZOOM_STEP : -ZOOM_STEP);
}, { passive: false });
```

修飾キーが押されていないときは早期returnしているため、通常のスクロールには影響しません。

# まとめ

AI Safe Markdown Previewにプレビューのズームイン・ズームアウト機能を追加しました。

- ボタン / `Ctrl`・`Cmd` + ホイールで操作。
- 50%〜300%を10%刻みで調整、パーセント表示クリックで100%にリセット。
- `font-size` スケーリングにより、テーブルやMermaidのレイアウトを崩さない。

VS Code Marketplaceからインストールできます。

https://marketplace.visualstudio.com/items?itemName=sakamoto-yoshitaka.vscode-robust-markdown

Claude CodeなどのAIエージェントとMarkdownを使う機会が多い方は、ぜひ試してみてください。
