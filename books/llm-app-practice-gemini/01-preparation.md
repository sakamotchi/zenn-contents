---
title: '事前準備'
---

# Python3 のインストール

Python3 をインストールしてください。

# VS Code のインストール

VS Code をインストールしてください。

## 拡張機能のインストール

下記の拡張機能をインストールしてください。

- Japanese Language Pack for Visual Studio Code
- Live Server
- Python
- Jupyter

# Python 仮想環境の作成

Python の仮想環境を作成します。

```bash
python3 -m venv .venv
```

下記のコマンドで仮想環境を有効化します。

```bash
. .venv/bin/activate
```

下記のコマンドで pip をアップグレードします。

```bash
python3 -m pip install --upgrade pip
```

下記のコマンドで必要なライブラリをインストールします。

```bash
python3 -m pip install -r requirements.txt
```
