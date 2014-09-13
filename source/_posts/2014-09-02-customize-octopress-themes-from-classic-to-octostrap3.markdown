---
layout: post
title: "Octopressのテーマをclassicからoctostrap3に変更してみる"
date: 2014-09-02 19:50:02 +0900
comments: true
categories: 
  - Octopress
---
## デフォルトテーマ
Octopressではデザインテーマを`install`タスクにてインストールします。デフォルトでインストールされるテーマは**classic**です。

![](/images/octopress_test_page.jpg)

## installタスクの動作を理解する

Octopressでテーマを適用する`install`タスクの動作を見てみようと思います。

下記コードはOctopressの`Rakefile`に記述された`install`タスクです。

```ruby
task :install, :theme do |t, args|
  if File.directory?(source_dir) || File.directory?("sass")
    abort("rake aborted!") if ask("A theme is already installed, proceeding will overwrite existing files. Are you sure?", ['y', 'n']) == 'n'
  end
  # copy theme into working Jekyll directories
  theme = args.theme || 'classic'
  puts "## Copying "+theme+" theme into ./#{source_dir} and ./sass"
  mkdir_p source_dir
  cp_r "#{themes_dir}/#{theme}/source/.", source_dir
  mkdir_p "sass"
  cp_r "#{themes_dir}/#{theme}/sass/.", "sass"
  mkdir_p "#{source_dir}/#{posts_dir}"
  mkdir_p public_dir
end
```

コードを見ても分かる通り、installタスクで実行していることは以下の6つの事だけで、とてもシンプルな動作です。`#{theme}`はデフォルトでは**classic**が代入される変数です(上記コードの6行目)。

 1. `source`ディレクトリを作成
 1. `.themes/#{theme}/source`ディレクトリ以下のファイルやディレクトリを再帰的に`source`ディレクトリにコピー
 1. `sass`ディレクトリを作成
 1. `.themes/#{theme}/sass`ディレクトリ以下のファイルやディレクトリを再帰的に`sass`ディレクトリにコピー
 1. `source/_posts`ディレクトリを作成
 1. `public`ディレクトリを作成


## テーマのインストール
テーマのインストールに必要な作業は以下の3つだけです。

1. `.themes`ディレクトリ以下にインストールしたいテーマをプロジェクトごとコピーする
   - `.themes/octostrap3/`のようになる
1. installタスクをインストールしたいテーマを明示して実行する
   - `rake 'install[octostrap3]'`のようになる
1. `generate`タスクを実行して静的ファイルを生成する

## octostrap3テーマをインストールする
今回は[Octostrap3](https://github.com/kAworu/octostrap3)をテーマとしてインストールしてみます。

#### octostrap3プロジェクトを`.themes`ディレクトリ以下にcloneしてきます。

```sh
git clone https://github.com/kAworu/octostrap3 .themes/octostrap3
```

`.themes`ディレクトリ以下はこのようなディレクトリ構成になるはずです。

<pre><code>### octostrap3テーマをローカル環境にcloneした状態
octopress
└── .themes/
    ├── classic
    └── octostrap3
</code></pre>

#### Octopressのinstallタスクをテーマを指定して実行

```sh
rake 'install[octostrap3]'
```

#### Octopressのgenerateタスクを実行して静的コンテンツを生成

```sh
rake generate
```

<br>
これでサイトのデザインテーマを`octostrap3`に変更できました！

![](/images/customize-octopress-themes_01.jpg)

<br>
### `と思ったらサイドサイドメニューのoctopress-tagcloud プラグインで生成されているulタグのデザインがoctostrap3に合っていない...`

![](/images/customize-octopress-themes_02.jpg)

<br>
これは[octopress-tagcloud](https://github.com/tokkonopapa/octopress-tagcloud)プラグインをインストールした際に置いたテンプレートファイルの書き方を修正することで対応できます。

## octopress-tagcloud プラグイン向けのHTMLテンプレートファイルを修正

プラグインを利用してコンテンツを生成している場合、`source/_includes/custom/asides`ディレクトリ以下にコンテンツ生成用のHTMLテンプレートファイルを配置していると思います。

今回、デザインテンプレートを`classic`から`octostrap3`に変更したので、追加したテンプレートファイルの`div`などのブロック構成、`class`や`id`などのデザインに関わるコードも`octostrap3`のデザインに合う様に修正しようと思います。

ちょうど、[octostrap3](https://github.com/kAworu/octostrap3)のブログページに[Category List Aside](http://kaworu.github.io/octopress/blog/2013/10/03/category-list-aside/)という記事でサイドメニューにカテゴリーリストを追加するためのテンプレートコードが掲載されていましたのでそのまま利用させて頂きます。

### 修正対象ファイル
以下のディレクトリツリーにある2つのHTMLファイルを修正します。

<pre><code>### octostrap3導入にあたって修正するテンプレートファイルのパス
octopress
└── source/
    └── _includes
        └── custom
            └── asides
                ├── category_list.html
                └── tag_cloud.html
</code></pre>

### 修正

##### source/_includes/custom/asides/category_list.html

```
<section class="panel panel-default">
  <div class="panel-heading">
    <h3 class="panel-title">Categories</h3>
  </div>
  <div class="list-group">
    {% for category in site.categories %}
    {% capture category_url %}/{{ site.category_dir }}/{{ category | first | slugize | downcase | replace:' ','-' | append:'/index.html'}}{% endcapture %}
    <a class="list-group-item {% if category_url == page.url %}active{% endif %}" href="{{ root_url | append:category_url }}">
        <span class="badge">{{ category | last | size }}</span>
        {{ category | first }}
      </a>
    {% endfor %}
  </div>
</section>
```

##### source/_includes/custom/asides/tag_cloud.html

```
<section class="panel panel-default">
  <div class="panel-heading">
    <h3 class="panel-title">Tag cloud</h3>
  </div>
  <span id="tag-cloud">{% tag_cloud counter:true %}</span>
</section>
```

<br>
簡単でした！


