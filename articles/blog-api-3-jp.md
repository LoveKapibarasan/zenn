---
title: "【自動化第3弾】note.com APIが死んだ日——HTMLスクレイピングへの転換と、全フォーマット投稿の完全実装"
emoji: "🚀"
type: "tech"
topics: ["api", "自動化", "go", "個人開発"]
published: true
---
![](/images/r2/app/blog-api/blog-api-3-jp.png)

前回（第2弾）では、Gistの表をNoteに埋め込むための「Iframelyの通行手形」を奪取する話をした。あれから数ヶ月、開発を続けるうちに**突然APIが沈黙した**。

```

GET /api/v1/text_notes/{id}  →  405 Method Not Allowed
```

記事の本文取得に使っていたエンドポイントが、ある日を境に一切応答しなくなった。Note.comの開発チームが静かにAPIを閉じたのだ。

> **注意：** 本記事は非公式APIの技術的な探求を記録したものです。サーバーへの過度な負荷を避け、規約を尊重した利用を前提としています。

---

## 1. 死亡確認：試みた全エンドポイント

パニックにならず、まず状況を整理した。

[https://gist.github.com/LoveKapibarasan/5474c7bc92d7183921a015a7787d1d18](https://gist.github.com/LoveKapibarasan/5474c7bc92d7183921a015a7787d1d18)

全滅だ。読み取り系のAPIはすべて封鎖された。しかし、記事のデータ自体は**公開ページとして存在している**。ならば答えは一つだ。

---

## 2. 転換：HTMLページをスクレイピングする

NoteのURLには記事ごとに一意のkeyが存在する。

```

https://note.com/{urlname}/n/{key}
```

このページをHTMLとして取得し、本文だけを抜き出せばいい。ブラウザのDevToolsでページを調べると、本文は必ずこの要素に格納されていた。

```html

<div data-name="body" class="note-common-styles__textnote-body">
  <p name="uuid" id="uuid">記事本文...</p>
  ...
</div>
```

Goの golang.org/x/net/html パッケージでDOMを走査し、data-name="body" を持つノードの子要素を丸ごと取得する実装に書き換えた。

```go

func extractNoteBodyHTML(pageHTML string) string {
    doc, _ := html.Parse(strings.NewReader(pageHTML))
    var find func(*html.Node) *html.Node
    find = func(n *html.Node) *html.Node {
        if n.Type == html.ElementNode {
            for _, attr := range n.Attr {
                if attr.Key == "data-name" && attr.Val == "body" {
                    return n
                }
            }
        }
        for c := n.FirstChild; c != nil; c = c.NextSibling {
            if r := find(c); r != nil { return r }
        }
        return nil
    }
    bodyNode := find(doc)
    var buf bytes.Buffer
    for c := bodyNode.FirstChild; c != nil; c = c.NextSibling {
        html.Render(&buf, c)
    }
    return buf.String()
}
```

これで41本の記事を全て逆抽出することに成功した。Gitリポジトリが一気に充実した瞬間だ。

---

## 3. 新発見：list APIのJSONはcamelCaseだった

逆抽出の過程でもう一つ問題が発覚した。ハッシュタグが全部空になる。

```yaml

tags:
    - ""
    - ""
    - ""
```

原因はJSON構造の思い込みだった。記事一覧API（/api/v2/creators/{urlname}/contents）のレスポンスは**camelCase**で返ってくる。

```json

{
  "publishAt": "2026-07-03T16:20:55+09:00",
  "likeCount": 4,
  "isLimited": false,
  "eyecatch": "https://...",
  "hashtags": [{ "name": "#TypeScript" }]
}
```

当初のGoの構造体は json:"publish_at" などsnake_caseで定義していた。修正後は正しくタグが取得できるようになった。

---

## 4. 修正：箇条書きが表示されなかった本当の理由

今回最も時間を食ったのが**リストのレンダリングバグ**だ。

投稿した記事を確認すると、箇条書きがただの文字列として並んでいる。NoteのHTML仕様を改めて解析すると、こういうことだった。

**誤った実装（動かない）:**

```html

<ul name="uuid" id="uuid">
  <li name="uuid" id="uuid">アイテムA</li>
</ul>
```

**正しい実装:**

```html

<ul name="uuid" id="uuid">
  <li>
    <p name="uuid" id="uuid">アイテムA</p>
  </li>
</ul>
```

自体にUUIDは不要。代わりに**内部の

にUUIDを付与**しなければならない。Goldmarkが生成した

でラップする後処理を追加した。

---

## 5. 表の完全自動化：Gist作成からEmbedまで

第2弾で手動でやっていた表の埋め込みを、完全に自動化した。

```

Markdown表を検出
  ↓
GitHub Gist APIでファイルを自動作成
  ↓
/api/v2/embed_by_external_api/check_type でURL検証
  ↓
/api/v2/embed_by_external_api でIframely HTML取得
  ↓
free_bodyに埋め込み
```

GITHUB_GIST_TOKEN を .env に設定するだけで、Markdown中の表が全自動でGist経由の埋め込みに変換される。

---

## 6. 画像の完全フロー

本文中の画像も自動化した。

```

![alt](local_path)  in Markdown
  ↓
POST /api/v3/images/upload/presigned_post  → S3署名付きURL取得
  ↓
POST {S3 action}  → 画像バイナリを直接S3にアップロード
  ↓
<figure name="uuid"><img src="{s3_url}" ...></figure>  に変換
```

eyecatch（見出し画像）は別途 POST /api/v1/image_upload/note_eyecatch で送信する。**注意点**: 本文にYouTube埋め込みがあると、NoteがYouTubeサムネイルをeyecatchとして自動設定してしまう。カスタムeyecatchは必ずPUT後にアップロードすること。

---

## 7. 現在の自動化フロー全体像

GitリポジトリをSSOT（Single Source of Truth）として、複数プラットフォームへ展開する構成に整理した。

```

notes/app/blog-api/blog-api-3-jp.md  ← SSOT
          │
    ┌─────┴──────┐
    ↓            ↓
note.com      Zenn.dev
(PUT API)    (articles/ 自動生成)
              │
              ↓
         R2 Storage (Cloudflare)
         └─ 画像の一元管理
```

フロントマターの設計：

```yaml

n_id: "12345678"       # note.com記事ID
q_id: "abc123"         # Qiita記事ID
title: "記事タイトル"
tags: [Go, API, 自動化]
private: false          # true → draft として投稿
eyecatch: ""            # note.comのeyecatch URL（投稿後に更新）
image_local_path: ""    # ローカル画像パス（eyecatch用）
image_urls: ""          # R2上のURL（Zenn eyecatch用）
slug: "slug-nXXXX"     # note.comのslug（初回公開後に変更不可）
edited: true            # 投稿フラグ（投稿後にfalseへ）
brushed_up: false       # ブラッシュアップ済みフラグ
skip_platform: []       # [note, zenn, qiita] で特定PFをスキップ
```

---

## 振り返り

APIが死ぬことは、開発者なら誰でも経験する。重要なのは**代替手段を見つける速度**だ。

今回の教訓：

- **Network タブは仕様書** — 公式ドキュメントがなくても、ブラウザが全てを教えてくれる
- **HTMLスクレイピングは最後の手段ではなく、正当な選択肢**
- **UUIDの付与ルールはタグごとに異なる** —
- は例外。思い込まずにブラウザで確認する
- **camelCaseとsnake_caseの混在** — 実際のレスポンスJSONを必ず目視確認する

次は Hatena Blog と Medium への展開を考えている。同じ「Network タブ解析」のアプローチで、どこまでいけるか楽しみだ。

---

**コメントや知見の共有はコメント欄へどうぞ。**
