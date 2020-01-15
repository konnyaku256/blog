---
title: "Welcome to my new blog !"
date: 2020-01-16T15:12:47+09:00
description: "私の新しいブログを紹介します。"
tags:
- poem
---

## ご挨拶
こんにちは、[@konnyaku256](https://twitter.com/konnyaku256)です。

このたび、新しくブログを開設してみました。
「こんにゃくの日記」という名前は、これまで投稿してきた[Medium](https://medium.com/%E3%81%93%E3%82%93%E3%81%AB%E3%82%83%E3%81%8F%E3%81%AE%E6%97%A5%E8%A8%98)と同じですが、内容はより技術的になるよう心がけたいと思っています。

今後、新しい「こんにゃくの日記」とMediumを共存させるかどうかは検討中です。
しばらく運用してみて、よさそうだったらMediumから完全に移行したいと思っています。
その際は、公開済みのStoryはすべてMediumに残し、Publications(こちらがMedium上のこんにゃくの日記に相当)のみ削除するようにしたいと考えています。

Mediumはとても書き心地がよく、記事を書くときのストレスがほとんどないのが気に入っていましたが、コードブロックや数式の拡張性が低いのが惜しいところでした。
今回の刷新で、このあたりの不満が解消されることを期待しています。

## こんにゃくの日記を支える技術
せっかくなので、簡単にこのブログの構成について紹介したいと思います。
リポジトリは[こちら](https://github.com/konnyaku256/blog)です。

このブログはGolang製の静的サイトジェネレータである[Hugo](https://gohugo.io/)を使って生成されています。
Hugoは
- Markdownで記事を書ける
- 高速ビルド
- ハイパフォーマンス
- 豊富なテーマ

が魅力です。

Lighthouseのスコア（特にチューニングしていなくても高いスコアが出ている）
![Lighthouseのスコア](/images/lighthouse.png)

ホスティングには[Netlify](https://www.netlify.com/)を使用しています。
GitHubにpushされたら自動でHugoのビルドが走ってホスティングされるようになっています。

それから、OGP画像の生成に[Cloudinary](https://cloudinary.com/)を使用しています。
このサービスのAPIを使用して動的に記事タイトルを画像化しています。

挿入するmetaタグの例
```html
<meta property="og:image" content="https://res.cloudinary.com/dmv5vdi93/image/upload/l_text:Sawarabi%20Gothic_50_bold:{{ .Title }},co_rgb:424242,w_700,c_fit/v1579096957/background.png">
```

## おわりに
ここまで読んでいただいてありがとうございます。
今後とも私とこのブログをよろしくお願いします。
