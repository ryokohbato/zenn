---
title: "Firefoxでも動くbackdrop-filterの代替実装"
emoji: "🦊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: false
---

この記事は[KMCアドベントカレンダー2021](https://adventar.org/calendars/6895)の19日目の記事です。大遅刻です。

# 概要

1月の半ば、[KMC](https://www.kmc.gr.jp) のSlack上でグラスモーフィズムが話題にあがりました。グラスモーフィズムは、すりガラスのように背景を透かしつつぼかすデザインです。[^1] ここで私は、グラスモーフィズムを素直に実装しても Firefox では動かないのではないか、という話を持ち出しました。

![それはそうと、グラスモーフィズム素直に実装するとFirefoxで動かない説があるんだけどもう対応してたっけ](https://storage.googleapis.com/zenn-user-upload/fd15a8c5a783-20220302.png)

![Can be enabled by setting the layout.css.backdrop-filter.enabled and gfx.webrender.all preference to true in about:config. ふむ](https://storage.googleapis.com/zenn-user-upload/08e0f9434031-20220302.png)

![一応黒魔術を使えば実装できたはず (忘れた)](https://storage.googleapis.com/zenn-user-upload/b9a899647a65-20220302.png)


実際にインターネットで検索してみると、CSS を用いたグラスモーフィズムの実装は `backdrop-filter` を使用する方法ばかりがヒットします。しかし、 `backdrop-filter` はデフォルト設定の場合 Firefox では動きません。[^2] [^3] そこで、`backdrop-filter` を使用せずとも同等の視覚効果を得る方法について考えます。

[^1]: [What is Glassmorphism? Create This New Design Effect Using Only HTML and CSS](https://www.freecodecamp.org/news/glassmorphism-design-effect-with-html-css/)
[^2]: ["backdrop-filter" | Can I use](https://caniuse.com/?search=backdrop-filter)
[^3]: [bug 1578503](https://bugzilla.mozilla.org/show_bug.cgi?id=1578503)
