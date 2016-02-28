---
title: Java とフロントエンドの付き合い方、それとチュートリアル
date: 2016-02-28 18:00:00
tags:
- java
- spring
- javascript
- browserify
- less
- task-runner
- make
---

ウェブアプリにおける JavaScript と CSS の役割・規模・複雑度が年々ヤバいことになってきているのは今更言うまでもない。今や JavaScript は、モジュールごとに分割して TypeScript や次期仕様の ECMAscript といったいわゆる altJS で記述されたのち、変換 (Transpile)・結合 (Concatinate)・最小化 (Minify) されてリリースされる。

ウェブアプリ開発において今時 (?) の言語を使う職場やコミュニティでは、フロントエンド絡みのビルドはある程度定着していることだろう。それには、[Ruby on Rails](http://rubyonrails.org/) における [Sprockets](https://github.com/rails/sprockets) のような、フロントエンド開発をサポートする優れたライブラリの存在によって、手軽に方法論を取り入れることができたり、コミュニティ内に情報が流通する機会がもたらされていることにも関係があると思う。


## Java のフロントエンド事情

Java ウェブアプリ開発とフロントエンド開発を統合するためのツールや情報は少ない。

あくまで自分のケースだが、周囲で見かける Java ウェブアプリのフロントエンド開発は、依然旧世代的だ。JavaScript にビルドという考えがあることを知らない開発者も多い。そんな現場での典型的なフロントエンド開発の運用は次のような感じになる。

* `<script src="common.js"></script>` から始まる `script` タグが10行以上並ぶ
* common.js からはグローバルネームスペースにいくつもの変数・関数がエクスポートされる
* common.js 以外に読み込んだページ固有スクリプトからもグローバルネームスペースへのエクスポートがある
* 一つの巨大な JavaScript や CSS ファイルができる

こうした運用は、コードの見通しを悪化させ、グローバル依存のコードを増やし、テストが放棄される。その結果、バグを生んでメンテナビリティを失う。

もちろん Java に限ってのことではないのだが、情報が少なくてなんとも切ない状況にある言語であるのは事実だ。


## シンプルなフロントエンド開発方法からの統合

じゃあ、と意気込んでフロントエンド開発事情を調べてみれば、それはそれで混沌としていることが分かるだろう。npm、Bower、Grunt、gulp、Browserify、webpack、……。何かいろいろあって、うまく使うと驚くほど効率的な開発ができるようになるらしい。だが、まずはシンプルに始めたい。それなのにシンプルに始める方法を選ぶのが難しい。

ずばり、シンプルさを重視すれば、[npm](https://www.npmjs.com/) と [Browserify](http://browserify.org/) だけで始めるのがいい。

まずは Java ウェブアプリのフロントエンド開発に npm と Browserify を使用するところから始めよう。その後はツールを追加して効率性を高めていこう。そんな開発ワークフローを体験するためのチュートリアルを作った。とにかく一度、フロントエンドのモダンな開発ワークフローを体験してもらいたい。


## Java + フロントエンド開発統合チュートリアル

チュートリアルでは次のサンプル Todo アプリを使用する。
* https://github.com/keik/spring-frontend-integration-example

サンプルの動作・開発には次の環境が必要になる。
* JDK 8+
* Apache Maven 3+
* Node 4+
* Gnu Make
* Unix-like シェル環境 (Windows の場合 MSYS や Cygwin で可能)

このチュートリアルは Node をインストールしたことがないレベルの初心者でも始められる。逆に説明が冗長だと感じたら飛ばしながら進むといい。


### 0. 段取り

チュートリアルは次の5つのステップに分かれており、サンプルアプリの各コミットに対応する。

1. ベースとなるサーバアプリを用意する ([ab4b918](https://github.com/keik/spring-frontend-integration-example/tree/ab4b918acb30ac49d8b2ee63e6d59432ce3d4e25))
2. フロントエンド開発用のファイルレイアウトとビルドタスクを作る ([f985e88](https://github.com/keik/spring-frontend-integration-example/tree/f985e88e0f925cb12bf013dd25e26b49c1cfd314))
3. 自動ビルドタスクを追加して効率化する ([948cd78](https://github.com/keik/spring-frontend-integration-example/tree/948cd78da0e418fc1d0e539275420942dd1db64b))
4. 自動ビルドを活用して JavaScript を実装する ([132f9a5](https://github.com/keik/spring-frontend-integration-example/tree/132f9a5d932e9679732d06f8311ffe5ef5b5a372))
5. CSS の自動ビルドタスクを作り、CSS を実装する ([1711f58](https://github.com/keik/spring-frontend-integration-example/tree/1711f586b309d88e721f90b357ee1a0811719cf3))


### 1. ベースとなるサーバアプリを用意する

*対応するコミット: [ab4b918](https://github.com/keik/spring-frontend-integration-example/tree/ab4b918acb30ac49d8b2ee63e6d59432ce3d4e25)*

Spring Boot を使った簡単な Todo アプリを用意した。なお、Spring Boot でサーバアプリを実装するところについてはチュートリアルの本目的ではないので省略する。この段階のファイルレイアウトは以下のようになっている。

```md
.
├── README.md
└── todo-app                         ... for sources of a Spring Boot server app
    ├── pom.xml
    └── src
        └── main
            ├── java
            └── resources
                ├── application.yml
                ├── static
                └── templates
```

Spring Boot は、組み込みサーバを起動してデプロイできる Maven ゴールが設定されている。次のコマンドで http://localhost:8080/ にデプロイされる。

```
% mvn spring-boot:run
```

まだ JavaScript は一切使用していない。以降の作業ではこれに JavaScript や CSS を加えてインタラクションと見栄えを整えていく。この段階での動作を確認しておくと以降のイメージがつかみやすいと思う。


### 2. フロントエンド開発用のファイルレイアウトとビルドタスクを作る

*対応するコミット: [f985e88](https://github.com/keik/spring-frontend-integration-example/tree/f985e88e0f925cb12bf013dd25e26b49c1cfd314)*

このステップでは、以下のファイルレイアウトになることを目指して作業を進めていく。

```md
.
├── README.md
├── todo-app                         ... for sources of a Spring Boot server app
│  ├── pom.xml
│  └── src
│      └── main
│          ├── java
│          └── resources
│              ├── application.yml
│              ├── static
│              │  └── bundle.js      ... built from todo-client/scripts/main.js
│              └── templates
└── todo-client                      ... for sources of client resources
    ├── Makefile
    ├── package.json
    └── scripts                      ... for sources of a JavaScript to bundle
        └── main.js                  ... entry point of JavaScript
```

フロントエンド開発に使用するモジュールは、Node パッケージマネージャ npm を使用してインストールしていくのが基本になる。npm は Node に同梱されている。まずサーバアプリ開発用の todo-app ディレクトリと同じ階層に、フロントエンド開発用の todo-client ディレクトリを作成し、そこで npm を初期化しよう。

```sh
% mkdir todo-client
% cd todo-client
% npm init
```

すると対話形式でプロジェクト情報の入力を求められるが、さほど重要ではないし後で修正もできるため、適当に入力する。これが済むと package.json ファイルが作られる。このファイルは、Maven における pom.xml のような、npm にとっての依存パッケージ情報などを管理するためのものだ。

次にこのプロジェクトで必要な npm パッケージをインストールする。モジュールごとに分割した JavaScript をビルドするために使用するツール Browserify は次のコマンドでインストールできる。

```sh
% npm install browserify --save-dev
```

`--save-dev` オプションをつけると、今インストールしたパッケージ名とバージョンが package.json 内に追記される。類似するオプションとして `--save` というものもあり、package.json に追記されるフィールドが異なる。使い分け方としては、`--save-dev` は開発時にのみ必要なツール (Browserify など) を、`--save` はランタイムに必要なライブラリ (jQuery や Bootstrap など) を指定する。この package.json ファイルがある場所で `npm install` コマンドを実行すると、package.json にある依存パッケージが自動的にまとめてインストールされるので、他のマシン環境でもコマンドを実行するだけで同じ依存パッケージを簡単に揃えることができる。

npm でインストールしたパッケージは node_moduels ディレクトリ内に格納される。Browserify のような、コマンドラインツールを提供しているパッケージの場合、node_modules/.bin 以下に実行ファイルが配置されるので、次のように Browserify を起動してみよう。

```sh
% node_modules/.bin/browserify -h
```

ヘルプが表示されるはずだ。さっそく JavaScript ファイルを作って Browserify で変換してみよう。

```sh
% mkdir scripts
% echo 'console.log("hi")' > scripts/main.js
% node_modules/.bin/browserify scripts/main.js
```

標準出力に表示されたものが変換後の JavaScript だ。

今度は npm で jQuery をインストールし、これを読み込むコードを書いてみよう。それを変換し、サーバアプリの静的ファイル置き場に出力してみよう。

```bash
% npm install jquery --save
% echo '
       var $ = require("jquery") // (1)
       console.log($().jquery)   // (2)
       ' > scripts/main.js
% mkdir ../todo-app/src/main/resources/static
% node_modules/.bin/browserify scripts/main.js -o ../todo-app/src/main/resources/static/bundle.js
```

こうしてビルドした bundle.js は、jQuery 本体のコード (1) と `console.log` で jQuery のバージョンを出力するコード (2) が結合された状態で出力される。これをサーバアプリの HTML テンプレート (todo-app/src/main/resources/templates/todos.html) で読み込むよう `script` タグを追加し、ページをリロードしてみよう。開発者ツールのコンソールに jQuery のバージョンが出力されるはずだ。

これが npm と Browserify による、最も基本的な JavaScript のビルド方法だ。

あとは楽をするために工夫をするフェーズだ。これまで `npm` や `mkdir` や `browserify` などのコマンドを、都度オプションを指定しながら直接実行してきた。これらのコマンド実行の流れを自動化する仕組みを Makefile で作っておくと便利だ。

```makefile
STATIC_DIR = ../todo-app/src/main/resources/static

all: clean $(STATIC_DIR)/bundle.js

$(STATIC_DIR)/bundle.js: node_modules $(STATIC_DIR)
	@node_modules/.bin/browserify scripts/main.js -o $@

$(STATIC_DIR):
	@mkdir -p $@

clean:
	@rm -rf $(STATIC_DIR)

node_modules: package.json
	@npm install

.PHONY: all clean
```

開発に途中参加する開発者は、ソースコードと package.json がコミットされたリポジトリをクローンして `make` さえすれば、他のコマンドを逐一実行しなくても、ビルドが完了するようになる。

なお Make でなくとも、シェルスクリプトやバッチファイル、Java のエコシステムを使いたいというのなら Gradle や Ant でもいい。だが Maven ですべてを統合するというのはやめておこう。Maven との格闘が始まり、決して効率的にならないだろう。[frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin) というのもあるが、タスクのカスタマイズが不自由になるのでオススメしない。


### 3. 自動ビルドタスクを追加して効率化する

*対応するコミット: [948cd78](https://github.com/keik/spring-frontend-integration-example/tree/948cd78da0e418fc1d0e539275420942dd1db64b)*

Browserify によって JavaScript をビルドすることはできたが、JavaScript ファイルを編集するたびに `browserify` を手動で実行し再ビルドするのは面倒だ。そこで今度は、ビルド対象およびビルド対象が読み込んでいるファイルが変更されたら、自動的にビルドしてくれるツール [watchify](https://github.com/substack/watchify) を導入しよう。

```sh
% npm install watchify --save-dev
```

watchify の使い方は Browserify とほぼ同じだ。基本的には自動ビルドするかしないかだけの違いしかない。

```sh
% node_modules/.bin/watchify scripts/main.js -o ../todo-app/src/main/resources/static/bundle.js -v -m
```

追加した `-v` オプションは、再ビルドの実行結果が標準出力するためのもので、動作状況を確認するために付けておいたほうが便利だ。`-d` オプションは、変換後のファイルに TODO sourcemap をバンドルするためのもので、これによって Chrome デベロッパーツールなどで結合前のファイルが表示できるようになるためデバッグに便利だ。

上のコマンドを実行した状態で、main.js に適当な編集を加えてみよう。自動的に再ビルドされ bundle.js が更新されるはずだ。再ビルドは思ったより早く終わることだろう。Node のストリームのパフォーマンスの素晴らしさに感動しよう。

このコマンドのエイリアスも、Makefile に追加しておこう。

```makefile
watch-js: node_modules $(STATIC_DIR)
	@node_modules/.bin/watchify scripts/main.js -o $(STATIC_DIR)/bundle.js -v -m
```

これで `make watch-js` を実行するだけで、インクリメンタルビルドが開始されるようになった。あとはガシガシ JavaScript を実装しよう。


### 4. 自動ビルドを活用して JavaScript を実装する

*対応するコミット: [132f9a5](https://github.com/keik/spring-frontend-integration-example/tree/132f9a5d932e9679732d06f8311ffe5ef5b5a372)*

このステップでは、以下のファイルレイアウトになることを目指して作業を進めていく。

```md
.
├── README.md
├── todo-app                         ... for sources of a Spring Boot server app
│  ├── pom.xml
│  └── src
│      └── main
│          ├── java
│          └── resources
│              ├── application.yml
│              ├── static
│              │  └── bundle.js      ... built from todo-client/scripts/main.js
│              └── templates
└── todo-client                      ... for sources of client resources
    ├── Makefile
    ├── package.json
    └── scripts                      ... for sources of a JavaScript to bundle
        ├── main.js                  ... entry point of JavaScript
        └── page-specifics           ... for store page-specific scripts
```

ステップ 3 で効率的にアプリを実装する準備ができたので、あとはひたすら実装する。Todo アプリの、Todo 追加・削除・完了を、AJax でサーバにリクエストするようにしよう。

このステップでのポイントは、ページ固有のスクリプトの実装・管理方法だ。

ページ固有スクリプトは、ページごとに分割して管理したい。最終的なリリースの仕方にはいくつかのパターンがあるが、今回はページ固有スクリプトも含めすべての JavaScript を一つのファイルにバンドルしてリリースすることにする。この場合、表示しているページ固有スクリプトだけが呼び出されるようにし、関係のないページ用スクリプトは呼び出されないようにする必要がある。これをうまくコントロールするためには、ページの `body` 要素の ID を参照するというアイデアが有効だ。

Todo 一覧ページ用の HTML テンプレート (todos.html) の `body` 要素には `page-todos` を付与しよう。

```html
<body id="page-todos">
```

そして Todo 一覧ページ用スクリプトは scripts/page-specifics/todos.js に作成し、その中で `body` の ID を参照して処理を呼び出すかどうかを判断するようにしよう。

```javascript
// todo-client/scripts/page-specifics/todos.js

if (document.body.id === 'page-todos')
  $(init)

function init() {
  // snips
}
```

これを main.js から読み込むようにする。Todo 一覧ページの他に新たなページが増えても、このように `body` の ID を参照させる方法で追加していけばいい。なお、ローカルのファイルを `require` する場合は、引数にファイルの相対パスを書き、`.js` は省略可能だ。

```javascript
// todo-client/scripts/main.js

require('./page-specifics/todos')
```

JavaScript のビルド環境・開発ワークフローについてはひとまずこれで完成だ。Ajax による Todo 操作の実装についてはただの jQuery を使った実装なので、サンプルのソースコードを確認してほしい。

ところで今回のサンプルでは、ページ固有スクリプトしか作成しておらず、共通関数モジュールを定義していない。もし共通関数などをモジュール化する場合には、エクスポートしたい関数・オブジェクトなどを `module.exports` に代入するようにする。例えば、`greet` という関数を持つオブジェクトをエクスポートする場合は、次のように記述する。

```javascript
// utils.js

module.exports = {
  greet: function() {
    console.log('hi')
  }
}
```

このような `module.exports` に代入のあるファイルを `require` で読み込むと、戻り値として代入されたオブジェクトや関数が得られる。よって次のように読み込んで使うことができる。

```javascript
// main.js

var utils = require('./utils')
utils.greet()
```

上で作成したページ固有スクリプトのように、`module.exports` に代入していないファイルを読み込んだ場合は、ファイルの頭から処理が呼び出される動作になる。

この `module` 変数や `require` 関数によるモジュール管理は Node で使われている方式であり、CommonJS と呼ばれる。


### 5. CSS の自動ビルドタスクを作り、CSS を実装する

*対応するコミット: [1711f58](https://github.com/keik/spring-frontend-integration-example/tree/1711f586b309d88e721f90b357ee1a0811719cf3)*

JavaScript と同じように、CSS の実装も効率化しよう。

このステップでは、以下のファイルレイアウトになることを目指して作業を進めていく。

```md
.
├── README.md
├── todo-app                         ... for sources of a Spring Boot server app
│  ├── pom.xml
│  └── src
│      └── main
│          ├── java
│          └── resources
│              ├── application.yml
│              ├── static
│              │  ├── bundle.js      ... built from todo-client/scripts/main.js
│              │  └── style.css      ... built from todo-client/styles/main.less
│              └── templates
└── todo-client                      ... for sources of client resources
    ├── Makefile
    ├── package.json
    ├── scripts                      ... for sources of a JavaScript to bundle
    │  ├── main.js                   ... entry point of JavaScript
    │  └── page-specifics            ... for store page-specific scripts
    └── styles                       ... for sources of a CSS to bundle
        ├── main.less                ... entry point of CSS
        └── page-specifics           ... for store page-specific styles
```

まず CSS をより便利に記述するための [Less](http://less-ja.studiomohawk.com/) を導入する。公式ウェブサイトの紹介にあるように、Less は CSS 上で変数や演算、入れ子による記述を可能にしたようなものだ。Less は npm からインストール可能だ。

```sh
% npm install less --save-dev
```

Less は `lessc` というコマンドラインツールを提供している。`lessc` に *.less ファイルを指定すると CSS が生成される。

そして Less のビルドも自動ビルド化したい。指定したファイルの変更を検知して、任意のコマンドを実行するパッケージ [chokidar-cli](https://github.com/kimmobrunfeldt/chokidar-cli) あるいは [watchf](https://github.com/keik/watchf) を導入しよう。chokidar-cli は有名だ。一方 watchf は変更されたファイル名をコマンドパートに埋め込むことができる。

```sh
% npm install watchf --save-dev
```

Makefile に Less の手動ビルド・自動ビルドのタスクを追加しよう。

```makefile
STATIC_DIR = ../todo-app/src/main/resources/static

# 手動ビルド
$(STATIC_DIR)/style.css: node_modules $(STATIC_DIR)
	@node_modules/.bin/lessc styles/main.less $@

# 自動ビルド
watch-less: node_modules $(STATIC_DIR)
	@node_modules/.bin/watchf "styles/**/*.less" -c "node_modules/.bin/lessc styles/main.less $(STATIC_DIR)/style.css"
```

これで `make watch-less` を実行すれば、Less が自動ビルドされるようになった。だが、`make watch-js` も同じような自動ビルドタスクだ。これを別の端末で起動させておくのは面倒だ。

よって、複数のタスクを並列実行する `watch` タスクも追加しよう。

```makefile
watch:
	$(MAKE) -j watch-js watch-less
```

これで `make watch` すれば JavaScript も Less も自動ビルドされるようになる。他に並列実行したいプロセスが追加されたら、Makefile の `watch` ターゲットに追加していけばいい。


## 99. 他に何ができるか

Java アプリとビルドを統合する際は、todo-app および todo-client ディレクトリのある階層に、`mvn -f todo-app/pom.xml` と `make -C todo-client` を実行するタスクを持った親 Makefile を配置すればいい。

より効率的な開発のために、npm でインストールできる他のツールも活用していってほしい。カテゴリごとに便利なライブラリを紹介して終わりにしたい。

### ミニファイ

リリースビルド時はミニファイをしたいので、JavaScript については [UglifyJS2](https://github.com/mishoo/UglifyJS2) を、CSS については [less-plugin-clean-css
](https://github.com/less/less-plugin-clean-css) を使うといい。

### コード変換

JavaScript のソースに altJS などを使いたければ Browserify 時に追加の変換処理を加えるトランスフォーム (Browserify プラグイン) を使うといい。TypeScript や ECMAScript、CoffeeScript などから JavaScript に変換するものをはじめ、主要なトランスフォームは https://github.com/substack/node-browserify/wiki/list-of-transforms にリストされている。

### コードチェック

Makefile にコードチェック (Lint) タスクを設定し、`all` ターゲットの依存に加えることで、コードチェックエラーがある場合にビルドを失敗させることができる。

コードチェックには [ESLint](http://eslint.org/) が機能性・使い勝手の両面からオススメだ。

### テスト

Makefile にテストタスクを設定し `all` ターゲットの依存に加えることで、テスト失敗時にビルドを失敗させることができる。

テストランナーは [mocha](https://mochajs.org/) や [tape](https://github.com/substack/tape) がシンプルでオススメだ。アサーションライブラリは [power-assert](https://github.com/power-assert-js/power-assert) が便利だが、セットアップに手こずるようなら [Chai](http://chaijs.com/) がシンプルだ。


## 次のステップ

クライアント MVC フレームワークや仮想 DOM やサーバサイドレンダリングが待ち受けている。そう考えるとフロントエンドやっぱり結構難しい。ただ、このような技術がどんな場合にも求められるようになるのは当分先だと思うので、それまでのレベルの底上げとしてメンテナブルなフロントエンド開発を浸透させていきたい。
