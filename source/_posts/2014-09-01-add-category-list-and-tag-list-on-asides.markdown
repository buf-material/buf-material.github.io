---
layout: post
title: "Octopressでカテゴリーリストとタグクラウドをサイドメニューに追加する"
date: 2014-09-01 19:40:34 +0900
comments: true
categories: 
  - Octopress
---

## デフォルトのサイドメニュー
Octopressをインストールした直後は、サイドメニューは`Recent Posts`のみです。

![](/images/add-category-list-asides_01.jpg)

<br>
そこで**Octopress向けに公開されているプラグインを使ってサイドメニューにカテゴリーリストとタグクラウドを実装**します。

## プラグイン
[jekyll](https://github.com/jekyll/jekyll)には、特定コンテンツをHTMLとして生成してあなたのサイトに表示させるための[プラグイン](http://jekyllrb.com/docs/plugins/)という仕組みを持っています。そして同時に、多くのプラグインが作成・公開されています。

[imathis/octopress](https://github.com/imathis/octopress)は[jekyll](https://github.com/jekyll/jekyll)のためのフレームワークであり、静的コンテンツ生成にはjekyllを使用しているため、もちろんプラグインを導入することが可能です。

## tokkonopapa/octopress-tagcloud プラグインのインストール
[tokkonopapa/octopress-tagcloud](https://github.com/tokkonopapa/octopress-tagcloud)のソースコードをcloneして、[jekyllプラグインインストール作法](http://jekyllrb.com/docs/plugins/#installing-a-plugin)に準じてファイルを配布します。

```sh
git clone git@github.com:tokkonopapa/octopress-tagcloud.git
cp -p octopress-tagcloud/plugins/tag_cloud.rb plugins/
cp -p octopress-tagcloud/source/_includes/custom/asides/* source/_includes/custom/asides/
```

jekyllの設定ファイル`_config.yml`にtokkonopapa/octopress-tagcloud プラグインで生成する静的ファイルをサイドメニューのコンテンツとして使用するための設定を記述します。

Octopressの`_config.yml`に`default_asides:`から始まる行があります。`default_asides:`はサイドメニューに表示するHTMLファイルを配列で記述していきます。

```yaml tokkonopapa/octopress-tagcloud configuration (_config.yml)
default_asides: [
    asides/recent_posts.html,
    custom/asides/category_list.html,
    custom/asides/tag_cloud.html
]
```

静的ファイルを生成してWebサーバを起動します。

```sh
rake preview

# GitHub Pages等でホスティングさせている場合はデプロイコマンドも実行しましょう
rake deploy
```

これでサイドメニューに以下画像のようなカテゴリーリストとタグクラウドが表示されたはずです。

![](/images/add-category-list-asides_02.jpg)

<br>
簡単でした!
