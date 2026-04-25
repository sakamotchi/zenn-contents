---
title: "Claude Codeの「許可待ち」をOSのネイティブ通知で受け取る仕組みをSDDeskに実装した"
emoji: "🔔"
type: "tech"
topics: ["Tauri", "AI", "ClaudeCode", "Rust", "デスクトップアプリ"]
published: true
publication_name: "third_tech"
---

:::message
**📢 プロダクト名変更のお知らせ**

2026-04-25 をもって、本プロダクトは `SpecPrompt` から **`SDDesk`** に改名しました。同名の別プロダクト（[specprompt.com](https://specprompt.com/)）との差別化のためです。

本記事中の旧名表記は新名に更新済みですが、記事 URL の slug は旧名（`spec-prompt-*`）のまま維持しています。
:::

# はじめに

仕様駆動開発のための軽量デスクトップアプリ「**SDDesk**」に、**Claude Code通知機能** を実装しました。Claude Codeの「許可待ち」「処理完了」「エラー」などをOSのネイティブ通知として受け取れるようになります。

https://github.com/sakamotchi/sddesk

SDDesk自体の概要は以下の記事を参照してください。

https://zenn.dev/sakamotchi/articles/spec-prompt-introduction

この記事では、なぜこの機能を作ったのか、どんな仕組みで通知を発火しているのかを紹介します。

# 解きたかった課題: Claude Codeの「待ち」を見逃す

Claude Codeで実装を任せていると、こんな場面が頻繁に発生します。

- **長めの作業をClaude Codeに依頼してブラウザで調べ物をしていたら、許可ダイアログでずっと止まっていた**
- **複数のターミナルタブで並行作業させていて、どのタブが完了したか分からない**
- **エラーで止まっていることに気づかず、しばらく経ってから戻ってきた**

ターミナルの中で起きているイベントは、ターミナルを見ていないと気づけません。アプリが非フォーカスのとき、あるいは別のタブを見ているときに、**「いまClaude Codeはあなたの応答を待っています」** とOSレベルで知らせてほしい。これが今回の機能のモチベーションです。

# 通知発火の2つの経路

SDDeskは2つの経路でClaude Codeのイベントを検出します。

## 経路1: OSC 9 エスケープシーケンス

`OSC 9`（Operating System Command 9）はターミナル向けの通知用エスケープシーケンスです。`ESC ] 9 ; <message> BEL` という形式で、多くのターミナルエミュレータが通知発火に使っています。

Claude Codeをはじめ、対応したCLIツールがこのシーケンスを出力すると、SDDeskのターミナルエンジン（alacritty_terminal）がイベントを受信し、ネイティブ通知を発火します。

## 経路2: ローカルHTTPフックサーバ

OSC 9に対応していないイベントや、より構造化されたペイロードを扱うために、**ローカルHTTPサーバ** をアプリ内で常駐させています。

- ループバックの `127.0.0.1:19823` でLISTEN
- `POST /claude-hook/{event}` でClaude Codeのhookペイロードを受信
- `tiny_http` クレートで実装し、Tauriの `setup()` で別スレッドとして起動

Claude Codeのhook機能から `curl` でこのエンドポイントを叩く設定を入れておけば、Permission（許可待ち）/ Completed / Error などのイベントを構造化された形で受け取れます。

ループバック限定でLISTENしているので、外部ネットワークには公開されません。

# どのタブから来た通知かを表示する

複数のターミナルタブで並行作業していると、「どのタブのClaude Codeが通知を出したのか」が分からないと意味がありません。

SDDeskの通知タイトルは次の形式で表示します。

```
Claude Code — {タブの表示タイトル}
```

タブの表示タイトル（例: `claude (my-project)`）はOSC 0/1/2で動的に更新されており、Rust側の `DisplayTitleCache` に `ptyId -> displayTitle` のマップで保持しています。フロント側で表示タイトルが変わるたびに `set_pty_display_title` コマンドを呼んでキャッシュを同期する設計です。

通知本文は、hookペイロードの `message` / `body` / `text` / `description` などから優先順で抽出します。Claude Codeのhookペイロード形式の揺れに対応するためです。

# フォーカス中は通知しない

「いま見ているタブの通知が画面右上にポップアップする」のは煩わしいだけなので、次の両方が成立するときだけ通知を抑止します。

1. アプリウィンドウがフォーカス中
2. 発火元タブがアクティブタブ

逆に言うと、別アプリを見ている、別タブを見ている、最小化している、いずれの状態でも通知は届きます。

# アプリ内の未読マークと連動させる

OS通知を出すだけでなく、アプリ内のタブにも未読マークを付けます。

- 通知発火時、Rustから `claude-notification-fired` イベントを `pty_id` 付きで `emit`
- フロントの `AppLayout` がlistenし、`terminalStore.markUnread(ptyId)` を呼ぶ
- 該当タブに **琥珀色（#F59E0B）のドット + 左ボーダー + 薄い背景** を付与
- ユーザーがそのタブをアクティブ化、またはウィンドウにフォーカスが戻ると自動で解除

OS通知は「気づき」のための一時的なもので、アプリに戻ってきたときに「**どのタブで何かあったか**」を視覚的に追える、二段構えの設計です。

# 通知発火フロー全体

ここまでの動きを一枚の流れにまとめると次のようになります。

```
[Rust] OSC 9 / HTTP hook を検出
   │
   ▼
[Rust] DisplayTitleCache から発火元タブ名を取得
   │
   ▼
[Rust → OS] ネイティブ通知送信（tauri-plugin-notification）
   │
   ▼
[Rust → Frontend] emit("claude-notification-fired", { pty_id })
   │
   ▼
[Frontend] terminalStore.markUnread(ptyId) で琥珀ハイライト付与
   │
   ▼
[Frontend] タブをアクティブ化 / ウィンドウ focus 復帰
   │
   ▼
[Frontend] terminalStore.clearUnread(tabId)
```

# 実装メモ: tauri-plugin-notification と osascript フォールバック

通知の送出には `tauri-plugin-notification` を使っています。本番ビルドではこれで問題なく動作しますが、**debugビルドでは権限のスコープの都合で通知が出ないケースがあった** ため、`osascript` を呼ぶフォールバックを入れています。

```rust
// 概念コード
match plugin_notify(title, body) {
    Ok(_) => {},
    Err(_) if cfg!(debug_assertions) => {
        // osascript -e 'display notification ...' でフォールバック
    },
    Err(e) => log::warn!("notification failed: {}", e),
}
```

`src-tauri/capabilities/default.json` に `notification:default` の権限を付与している点も忘れがちなポイントです。

# 通知のON/OFFを設定で持つ

集中して作業したいときは通知が邪魔になることもあるため、**設定画面でON/OFFを切り替え** られるようにしました。

- デフォルト: 有効
- 設定値は `~/.config/sddesk/config.json` の `appearance.notification_enabled` に永続化
- アプリ再起動後も設定が引き継がれる

# Claude Codeのhook側設定例

Claude Code側のhook設定でローカルHTTPサーバを叩くようにすれば、OSC 9に依存せずより細かいイベントを受け取れます。設定ファイル（例: `.claude/settings.json`）にこんな形でhookを定義します。

```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "curl -s -X POST http://127.0.0.1:19823/claude-hook/notification -H 'Content-Type: application/json' -d \"$CLAUDE_HOOK_PAYLOAD\""
          }
        ]
      }
    ]
  }
}
```

これでClaude Codeが許可待ち・完了・エラーなどのフックを発火するたびにSDDeskに届きます。

# おわりに

「Claude Codeに任せて別のことをする」というスタイルが定着するほど、**処理の節目で気づける通知** の重要性が増してきました。OSC 9とHTTPフック、二経路で取りこぼしを減らし、タブ名差し込み・未読マーク連動まで含めて、SDDeskの「軽量さを保ったまま実用度を上げる」方針に沿った機能になったと思います。

フィードバックや要望はGitHubのIssueでお待ちしています。

https://github.com/sakamotchi/sddesk
