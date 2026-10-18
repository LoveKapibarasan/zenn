---
title: "【自動化第6弾】もう二度とブラウザを操作しない。６つのプラットフォームへの統合をすべて自動化。"
emoji: "🚀"
type: "tech"
topics: ["自動化", "gitops", "api", "bluesky", "はてなブログ"]
published: true
---
![](/images/r2/app/blog-api/automation-6platforms-en.png)

## きっかけ

記事を書くたびに、同じ操作を6回繰り返していた。

Note、Qiita、Zenn、Medium、はてなブログ、Bluesky。それぞれのサイトを開き、コピペして、タグを設定して、画像をアップロードして、投稿ボタンを押す。

一本の記事を6か所に上げるだけで、30分以上かかることもある。

**これは完全に無駄だ。** 一度書いた内容を、なぜ6回手で操作しなければいけないのか。

そう気づいた日から、すべてを自動化することにした。

---

## 構成：GitリポジトリをSSOTとして

すべての記事は **Markdownファイル＋YAMLフロントマター** としてGitリポジトリに保存している。

```yaml
title: 記事タイトル
tags: [タグ1, タグ2]
n_id: "123456"       # Note記事ID
q_id: "abc123"       # Qiita記事ID
m_id: "50ce4aa5"     # Medium記事ID
h_id: "1494577..."   # はてなブログ記事ID
bsky_uri: "at://..."  # Bluesky投稿URI
edited: true          # trueのときだけ投稿対象
```

**Single Source of Truth（SSOT）**。一つのファイルが、すべてのプラットフォームの「正」のデータになる。

記事を書いたら edited: true にする。あとはコマンドを一つ叩くだけ。

---

## ６つのプラットフォームの自動化

### 1. Note.com — REST API

Note の非公式 API をリバースエンジニアリングして Go クライアントを実装。記事の作成・更新（Upsert）、アイキャッチ画像アップロードまで対応。

```bash
go run ./src note notes/app/whatcha/whatcha-2-0-jp.md
```

### 2. Qiita — REST API ＋ Playwright フォールバック

Qiita は公式 REST API がある。ただし新規作成は 1日 N 件制限があるため、ブラウザ自動化（Playwright）でも投稿できる二段構え。

```bash
go run ./src qiita-publish notes     # API経由（UPDATE）
python scripts/qiita_browser.py      # ブラウザ経由（CREATE）
```

### 3. Zenn — ローカル同期

Zenn は Git リポジトリ連携。Markdown を所定フォーマットに変換して articles/ ディレクトリに書き出すだけで公開される。

```bash
go run ./src zenn ../articles
```

### 4. Medium — Playwright

Medium は書き込み API が存在しない。Playwright でエディタを操作し、ステルスモードでBot検知を回避。

```bash
python medium_post.py notes/app/whatcha/whatcha-2-0-en.md
```

### 5. はてなブログ — AtomPub API

はてなブログは古典的な AtomPub API をサポート。XML over HTTP でブログ記事の CRUD が可能。HTML形式でコンテンツを管理し、デコレーション（スタイル付き HTML）も自動生成。

```bash
go run ./src hatena-sync --auto      # 既存記事をローカルに同期
```

### 6. Bluesky — AT Protocol

Bluesky は AT Protocol という分散型プロトコルを採用。App Password で認証し、リンクカード（外部埋め込み）＋画像 Blob アップロードまで API で完結。

```bash
go run ./src bluesky-post notes/app/whatcha/whatcha-2-0-en-short.md
```

---

## エンゲージメントも自動化

投稿するだけでは足りない。読んでもらうには、自分も積極的に他の人の記事にいいね・コメントする必要がある。

これも全自動化した。OpenAI GPT-4o-mini でコメント草案を生成し、YAMLキューファイルに保存。確認してから一括実行する。

https://gist.github.com/LoveKapibarasan/b9580258c5ba57c7cc5851b4db81f4ea

---

## コードの全体像

```
API/
  src/              # Go コマンド群
    post.go         # エントリポイント・FrontMatter定義
    note_post.go    # Note投稿
    hatena_sync.go  # はてなブログ同期
    bluesky_sync.go # Bluesky投稿・同期
    bluesky_engage.go # Blueskyエンゲージ
    engage.go       # Noteエンゲージ
    qiita_engage.go # Qiitaエンゲージ
    zenn_engage.go  # Zennエンゲージ
  pkg/
    hatena-client/  # AtomPub クライアント
    bluesky-client/ # AT Protocol クライアント
    medium-client/  # Medium API クライアント
    note-client/    # Note 非公式APIクライアント
    qiita-client/   # Qiita REST APIクライアント
    zenn-client/    # Zennローカル変換
  scripts/          # Python補助スクリプト
    medium_post.py      # Medium Playwright投稿
    medium_engage.py    # Medium Playwrightエンゲージ
    hatena_engage.py    # はてなスターエンゲージ
    qiita_browser.py    # QiitaブラウザCREATE
  notes/            # 記事ソース（SSOT）
```

---

## やってみてわかったこと

**プラットフォームごとにAPIの思想が全然違う。**

- Note → 非公式APIをリバースエンジニアリング
- Qiita → ちゃんとした REST API があるが制限あり
- Zenn → Git が API 代わり
- Medium → Bot検知が強く、Playwright + stealth必須
- はてなブログ → AtomPub という2005年の規格が現役
- Bluesky → 分散型プロトコル（AT Protocol）で最もクリーン

**それでも、一度作れば永続的に動く。** 30分かかっていた作業が、コマンド一つになった。

---

## 次の展望

**Instagram、Facebook、LinkedIn** への展開。

これらはいずれも OAuth ベースの API があるが、個人利用でのアクセス制限が厳しい。特に Instagram は Graph API の申請が必要。それでも自動化できる範囲は広い。

---

**「書くことに集中する。配信はシステムに任せる。」**

それがこの自動化の目標だった。

コードは全て GitHub で管理している。
