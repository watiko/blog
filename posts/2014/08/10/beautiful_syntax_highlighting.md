---
title: コードを綺麗に表示する highlight.js
author: watiko
type: post
date: 2014-08-09T18:35:12+00:00
url: /2014/08/10/beautiful_syntax_highlighting/
pdrp_attributionLocation:
  - end
categories:
  - 未分類

---
  
もともと `google code prettify` を使っていたのですが、なんとなく他のものを試していたら良さそうなのを見つけたので紹介。

highlightjs
===========  

[<img class="alignnone size-large wp-image-239" src="/image/2014/08/7db19e046cba7d7f6cfde466011373e6-1024x642.png" alt="highlightjs web" width="474" height="297" srcset="/image/2014/08/7db19e046cba7d7f6cfde466011373e6-1024x642.png 1024w, /image/2014/08/7db19e046cba7d7f6cfde466011373e6-300x188.png 300w, /image/2014/08/7db19e046cba7d7f6cfde466011373e6.png 1034w" sizes="(max-width: 474px) 100vw, 474px" />][1]

[highlight.js](https://highlightjs.org/ "highlight.js Syntax highlighting for the Web")

まずは、開いて `language` や `style` をポチポチしてみてください。どんな感じか解ると思います。

設定できるすべての項目が選べるわけではないので、「( ・∀・)ｲｲ!!」と思ったなら
  
[このページ](https://highlightjs.org/image/test.html "live demo")でいろいろ試してみるのがいいと思います。

<!--more-->

導入
====  

まずはスタイルシート(css)とスクリプト(js)を用意します。

- [Getting highlight.js](https://highlightjs.org/download/ "Getting highlight.js")

上記のページでハイライトする言語を設定できるので好みの設定にしたら`Download`をポチッとします。
  
解凍(伸張)してできたファイルから`highlight.pack.js`とお好みの`style`のスタイルシートを読み込みます。

とりあえず試してみたい場合などは、CDNが用意されているのでありがたく使いましょう。
  
(スタイルシートの`default`という部分を有効にしたい`style`に置き換えることでその`style`が使えます。ex: `monokai_sublime`)

```html 
<link rel="stylesheet" href="http://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.1/styles/default.min.css">
<script src="http://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.1/highlight.min.js"></script>
```

さて、これで必要なファイルはてにはいりましたが、これだけではハイライトされません。

- [How to use highlight.js](https://highlightjs.org/usage/ "How to use highlight.js")

用意したスクリプトを読み込んだあとに初期化をしなければいけないので、すぐ下に下記のコードを挿入します。

```html
<script>hljs.initHighlightingOnLoad();</script>
```

さて、これでもまだハイライトされません。さらに、ハイライトしたいコード側も特定の形式になっている必要があります。`html`をハイライトする場合は、

```html
<pre><code class="html">...</code></pre>
```

こんな感じにマークアップされている必要があります。適切にマークアップされていればもうハイライトが有効になっているはずです。

もし、既存のページに導入しようとしていて、マークアップを変更するのが難しい場合は上記の`How to use highlight.js`にJQueryを使ってハイライトを有効にする要素を変更する方法が記載されているのでそれを参考にするといいと思います。(試していないので具体的にはどうするかはあまり把握していません)
  
[1]: https://highlightjs.org/
