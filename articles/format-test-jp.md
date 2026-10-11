---
title: "【テスト投稿】note.com全フォーマット動作確認"
emoji: "🚀"
type: "tech"
topics: ["テスト", "個人開発"]
published: true
---
> この記事はnote.com投稿システムのフォーマットテスト用です。

---

## 1. 見出し

## 見出しH2

### 見出しH3

---

## 2. テキスト装飾

通常テキスト。**太字（Bold）**。*イタリック（Italic）*。***太字イタリック***。

インラインコード: const x = 42

---

## 3. 箇条書き・番号リスト

**箇条書き（Unordered）:**

- アイテム A
- アイテム Bネスト B-1ネスト B-2
- アイテム C

**番号付きリスト（Ordered）:**

ステップ1
ステップ2
ステップ3

---

## 4. コードブロック

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}
```

```bash
# Dockerコンテナ起動
docker compose up -d
docker ps
```

---

## 5. 引用

> 「シンプルであることは、複雑であることよりも難しい。」— スティーブ・ジョブズ

---

## 6. 水平線

上のコンテンツ

---

下のコンテンツ

---

## 7. リンク

[note.com](https://note.com) — 公式サイト

---

## 8. YouTube埋め込み

@[youtube](dQw4w9WgXcQ)

---

## 9. インライン画像

![](https://assets.st-note.com/img/1783860273-hIE3Jypi642F7oKf8X5aSbDj.png?width=1200)

---

## 10. 表（Gist埋め込み）

https://gist.github.com/LoveKapibarasan/867154fa392c012f1baf70f9c8018c13

---

## 11. まとめ

このテスト投稿でnote.comの全主要フォーマットの動作を確認しました。
