---
layout: post
title: "Octopress + GitHub Pages = Simple Blog"
date: 2014-07-20 23:49:28 +0900
comments: true
categories: 
  - Octopress
  - GitHub Pages
  - Blog
---

いきなりですが、ブログを開設したいと思いました。

VimやMarkdownの勉強も兼ねることができるので、**Octopress + GitHub Pages**で書いてみることにします。

## Octopress

http://octopress.org/

![](/images/octopress_top.jpg)

**[Octopress](http://octopress.org/)**とは、[mojombo/jekyll](https://github.com/jekyll/jekyll)という静的ファイルジェネレータのためのフレームワークです。

**jekyll**がMarkdownでの記事作成に対応しているのでエンジニアの方で利用されている方が多いようです。

  1. HTML5,レスポンシブデザイン対応
    - デフォルトのテンプレートデザインで既にレスポンシブデザイン対応
  1. Compass,Sassを用いたデザイン
  1. 記事の新規作成やローカルレビューなどの操作はrakeタスクに纏まっていてシンプル
  1. ローカルで作成した記事を反映させる方法は、`Github pages`や`Rsync`を利用
    - `Github pages`の場合はgithubへpushして更新
  1. `POW`,`WEBrick`,`thin`などのRackサーバをサポート
  1. カラースキームとして[solarized](http://ethanschoonover.com/solarized)を採用することで美しいシンタックスハイライトを実現
  1. Markdownで記事を作成可能

## GitHub Pages

https://pages.github.com/

![](/images/github_pages_top.jpg)

**[GitHub Pages](https://pages.github.com/)**とは、`username.github.io`というレポジトリに静的コンテンツを置くことでWebサイトとして公開できるものです。
WebサイトのデフォルトURLはレポジトリ名と同じ`http://username.github.io`となります。
ちなみに、`username`は自分のGitHubアカウント名に置き換えてください。

**GitHub Pages**としてWebページを作成することでコンテンツをGit管理にできるほか、GitHubレポジトリに置けることでバックアップとしての意味合いも果たします。

そして、**GitHub Pages**では静的ファイルジェネレータの[mojombo/jekyll](https://github.com/jekyll/jekyll)を使ったブログ作成が可能なので、**Octopress**もインストール可能なのです。

## Octopressをローカル環境にインストール

今回の最終目標は**GitHub Pages**でOctopressのエントリー記事を表示させることですが、記事作成などの主な操作はローカル環境で行う必要があります(GitHubレポジトリの内容を直接操作することはできない)。ローカル環境で記事作成などの操作を行った後に`GitHubへPushする`ことで**GitHub Pages**に反映させるのです。

Octopressのソースコードを`git clone`でローカル環境に持ってきます。
ローカル環境側のRubyバージョンが`1.9.3`以上の必要があるので、[rbenv](https://github.com/sstephenson/rbenv)や[rvm](https://github.com/wayneeseguin/rvm)などでインストールしておきます。


```sh
git clone git@github.com:imathis/octopress.git username.github.io
cd username.github.io
```

gemモジュール群をインストールします。私の環境では、`Could not find a JavaScript runtime`エラーが発生するのを防ぐために、**therubyracer**プラグインも一緒にインストールします。

`Gemfile`に追記します。

```ruby Gemfile
# for "Could not find a JavaScript runtime" Error
gem 'therubyracer'
```

`Gemfile`に記述されているGemモジュール、および依存関係のあるGemモジュールをインストールします。

```sh
bundle install
```

`rake install`コマンドを実行して**Octopress初期セットアップとしてデフォルトテーマ用のファイルをJekyllの静的コンテンツ生成パスにコピー**します。
rakeタスク実行時の標準出力をそのまま記載しているので`mkdir`や`cp`はrakeタスク内で実行済みです。


```sh
$ bundle exec rake install
## Copying classic theme into ./source and ./sass
mkdir -p source
cp -r .themes/classic/source/. source
mkdir -p sass
cp -r .themes/classic/sass/. sass
mkdir -p source/_posts
mkdir -p public
```

この段階で初期ページの表示確認は可能なはずですので`rake preview`コマンドを実行して、ローカル環境でWebページを公開します。

`rake preview`コマンドを実行することで**Rakefile内のpreviewタスクを実行**します。previewタスクでは`jekyll build --watch`コマンドを実行して静的コンテンツをジェネレートしつつ、以降の変更を監視し続けます。変更を検知すると再ジェネレートを自動実行してくれます。そして`rackup`コマンドを実行し、TCPポート4000番でWEBrickサーバを起動します。

ここでも`rake preview`タスクの実行時の標準出力を(_config.ymlパス以外は)そのまま記載します。

```sh
$ bundle exec rake preview
Starting to watch source with Jekyll and Compass. Starting Rack on port 4000
[2014-07-22 01:08:53] INFO  WEBrick 1.3.1
[2014-07-22 01:08:53] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-07-22 01:08:53] INFO  WEBrick::HTTPServer#start: pid=4585 port=4000
Configuration file: /path/to/username.github.io/_config.yml
>>> Change detected at 01:08:54 to: screen.scss
            Source: source
       Destination: public
      Generating... 
                    done.
 Auto-regeneration: enabled
identical public/stylesheets/screen.css 
```

ブラウザで`http://localhost:4000`(localhostの部分はローカル環境のIPアドレスに置き換えてください)にアクセスすると以下のような初期ページが確認できます。

![](/images/octopress_initial.jpg)

## Octopressで指定可能なrakeタスク

Octopressをインストールする際に`rake install`コマンドや`rake preview`コマンドを実行しましたが、**rakeコマンドの後ろに指定したサブコマンドはRakefileに定義されたタスク**に対応しています。

指定可能なrakeタスクは`rake -T`コマンドで確認できます。

```sh
$ bundle exec rake -T 
rake clean                     # Clean out caches: .pygments-cache, .gist-cache, .sass-cache
rake copydot[source,dest]      # copy dot files for deployment
rake deploy                    # Default deploy task
rake gen_deploy                # Generate website and deploy
rake generate                  # Generate jekyll site
rake install[theme]            # Initial setup for Octopress: copies the default theme into the path of Jekyll's generator
rake integrate                 # Move all stashed posts back into the posts directory, ready for site generation
rake isolate[filename]         # Move all other posts than the one currently being worked on to a temporary stash location (stash) so regenerating the site happens much more quickly
rake list                      # list tasks
rake new_page[filename]        # Create a new page in source/(filename)/index.markdown
rake new_post[title]           # Begin a new post in source/_posts
rake preview                   # preview the site in a web browser
rake push                      # deploy public directory to github pages
rake rsync                     # Deploy website via rsync
rake set_root_dir[dir]         # Update configurations to support publishing to root or sub directory
rake setup_github_pages[repo]  # Set up _deploy folder and deploy branch for Github Pages deployment
rake update_source[theme]      # Move source to source.old, install source theme updates, replace source/_includes/navigation.html with source.old's navigation
rake update_style[theme]       # Move sass to sass.old, install sass theme updates, replace sass/custom with sass.old/custom
rake watch                     # Watch the site and regenerate when it changes
```

## 記事を投稿する

新しい記事を投稿するためには`rake new_post[title]`コマンドを実行します。`title`部分は任意のタイトル文字を記述します。英字以外だとうまく認識できないケースが考えられるので、ここでは英字で指定します。

```sh
rake new_post[test_post]
```

`rake new_post[test_post]`コマンドで`source/_posts/yyyy-mm-dd-test-post.markdown`というファイルが作成されます。


```sh
vim source/_posts/yyyy-mm-dd-test-post.markdown
```

テストのために本文を１行だけ追記しました。

```text source/_posts/yyyy-mm-dd-test-post.markdown
---
layout: post
title: "test_post"
date: yyyy-mm-dd 00:00:00 +0900
comments: true
categories: 
---

テスト投稿です
```

静的ページを生成します。

```sh
rake generate
```

サーバを立ち上げます。

```sh
rake preview
```

ブラウザで`http://localhost:4000`(localhostの部分はローカル環境のIPアドレスに置き換えてください)にアクセスします。

![](/images/octopress_test_page.jpg)

## OctopressをGithub Pagesで表示させる

今までローカル環境で編集してきたOctopressコードを新規作成した記事と共にGitHubにpushします。

### GitHubレポジトリ作成


GitHubの新規レポジトリ作成ページにアクセスします。

https://github.com/new

`Repository name`に`username.github.io`(usernameは自分のGitHubアカウント名)を指定し、`Create repository`ボタンをクリックします。

![](/images/create_new_repo.jpg)

### ローカル環境にてgitレポジトリの作成

**OctopressでGitHub Pagesを利用する場合のgitレポジトリ作成には`setup_github_pages`という専用のrakeタスクが用意されています。**

`rake setup_github_pages`コマンドを作成したレポジトリのURLと共に実行します。

  - レポジトリのURL例
    - `git@github.com:your_username/your_username.github.io.git`
    - `https://github.com/your_username/your_username.github.io`

```sh
rake setup_github_pages[git@github.com:username/username.github.io.git]
```

### GitHubへpush

Octopressで記事などをGitHubへpushする時には`deploy`タスクを利用します。

```sh
rake deploy
```

GitHubにpush後10分程度待つと、`http://username.github.io`で先ほどまでローカル環境で表示できていたOctopressページを表示させることに成功しました。

