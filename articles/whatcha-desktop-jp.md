---
title: "Whatcha が Mac / Windows / Linux で使えるようになりました — そして 15 言語対応へ"
emoji: "🚀"
type: "tech"
topics: ["日記", "デスクトップ", "flutter", "i18n", "microsoftstore"]
published: true
---
![](/images/r2/app/whatcha/whatcha-desktop-jp.png)

## Whatcha は、いつでも、どの画面からでも書ける日記に。

![](https://assets.st-note.com/img/1786472704-DR6QOrBkfxTJscVlCUyiPgSZ.png?width=1200)

これまで Whatcha は **iOS / Android / Web** の 3 プラットフォームで動いていました。この夏、そこに **macOS / Windows / Linux** の 3 つが加わりました。

朝は Mac で書き、昼は iPhone で写真を追加し、夜は Windows のノート PC で振り返る。そんな使い方が全部同じアカウントで自然にできる状態になっています。

---

## なぜデスクトップ版が必要だったか

日記アプリの「一日の中で使う時間」は、実はスマホより PC に向いている瞬間の方が多いのではないか、とずっと思っていました。

- 長い文章を書きたい日 → **キーボード**
- 旅行の写真をまとめてアップロードする日 → **カメラを取り込む PC の方が早い**
- 一年前の記録を落ち着いて読み返す日 → **大きい画面**

「思い出をみんなで一冊の本のように育てる」というコンセプト自体が、スマホの縦スクロールだけで完結するものではありませんでした。Whatcha を長く使う人ほど、机の前で腰を据えて開ける場所が欲しくなる。

だから今回、3 つの OS すべてで動く形にしました。

---

## macOS 版（Mac App Store）

Apple Silicon / Intel 両対応、macOS 12 以降。

Mac 版で一番時間がかかったのは、実は**ウィンドウの初期サイズ**でした。macOS だけ他プラットフォームと違って初期サイズが 800×600 で開いてしまい、Sign In 画面のワイドレイアウトが崩れる問題があって、1280×800 に修正しています。

Apple Sign In / Google Sign In もネイティブ実装で動くようになっています。

![](https://assets.st-note.com/img/1786472705-p3IRD2fUC8vWTeX7aEFmMrsu.png?width=1200)

---

## Windows 版（Microsoft Store）

Windows 10/11 対応、MSIX パッケージで配信。

Windows は WebView2 まわりで一番苦労しました。Cloudflare Turnstile（ログインの Bot 対策）が、loadData(baseUrl) 経由だと Origin が opaque になって動かないという癖があって、専用の HTTPS ページ（turnstile.html）を用意して JS ブリッジ経由でトークンを受け取る方式に落ち着いています。

Google OAuth も Windows 用の loopback 実装（google_sign_in_all_platforms）を追加。Apple Sign In は Windows で公式サポートが無いため非表示にしてあります。

Microsoft Store への提出も 15 言語ぶんのメタデータを Partner Center に Playwright で流し込む自動化スクリプトを組んで、丸ごと自動化しました。

![](https://assets.st-note.com/img/1786472707-MHjvqAlbVxCKGhsN0aiSWped.png?width=1200)

---

## Linux 版（OBS 経由で 12 ディストリ）

**Ubuntu 22.04 / 24.04 / 25.04 / 26.04、Debian 12 / 13、Fedora 43 / 44、openSUSE Tumbleweed / Leap 15.6 / 16.0** — 合計 12 ターゲット。

Linux は openSUSE の **OBS (Open Build Service)** を使って、DEB / RPM の両方を各ディストリごとに自動ビルドしています。ソースパッケージには Flutter で事前ビルドしたバイナリを含めて OBS 側では再ビルドしない構成にしたので、依存の解決がシンプルです。

flutter_inappwebview_linux が WPE WebKit を要求してくる問題は、Linux では Turnstile 自体を使っていないので依存を上書きで空にする形で回避しました。

![](https://assets.st-note.com/img/1786472707-pgcv3m6w4Vl1nLT2XAioHeOK.png?width=1200)

---

## 15 言語対応 — アラビア語からベトナム語まで

デスクトップ 3 OS 対応と並行して、**アプリ本体と Store の商品ページを 15 言語**に拡張しました。

**追加された言語:**アラビア語、ドイツ語、スペイン語、フランス語、ヒンディー語、インドネシア語、イタリア語、韓国語、ポルトガル語（ブラジル）、ロシア語、トルコ語、ベトナム語、中国語（簡体字）

（既存: 英語、日本語）

Play Store・App Store・Microsoft Store それぞれに、15 言語ぶんの Title / Short Description / Full Description / Changelog を投入しています。翻訳は AI 生成 + 現地の言い回しを Wikipedia や辞書で確認する流れで、機械翻訳感が薄まるよう微調整しました。

「思い出を、大切な人と一緒に育てる」というコンセプトは、家族の在り方や記憶に対する感覚が文化によってかなり違うので、直訳ではなく**そのニュアンスが伝わる訳語**を選ぶのに一番時間がかかりました。

例えば heroTagline（トップの一言）を各言語で見比べると、同じことを言っているのに響きがぜんぜん違って面白いです。

---

## これで揃ったこと

https://gist.github.com/LoveKapibarasan/301ecb3a51d8c27727fde0526ba19c84

同じアカウント・同じデータで、**朝は Mac、昼はスマホ、夜は Linux のデスクトップ**、というような使い方が現実的にできる形になりました。

---

## これから

書き終えたエントリの共同編集や、コミット履歴（Git 風のバージョン管理）の可視化まわりをデスクトップならではの UI で磨き込んでいく予定です。

大きい画面で開いた時にちゃんと嬉しい日記アプリを、少しずつ育てていきます。

**思い出は、一緒に育てていくもの。**

- サイト: [https://whatareyoudo.ing/](https://whatareyoudo.ing/)
- Mac App Store: [Whatcha](https://apps.apple.com/app/whatcha-diary/id6749620443)
- Microsoft Store: 審査通過次第公開
- Linux (openSUSE OBS): home:LoveKapibarasan:whatcha
