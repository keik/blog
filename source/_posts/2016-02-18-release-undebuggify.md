---
title: undebuggify リリースした
date: 2016-02-18 23:18:57
tags:
- javascript
- browserify
- transform
- ast
---

[debug](https://github.com/visionmedia/debug) を使用しているコードを AST を辿って削除する Browserify Transform モジュール [undebuggify](https://github.com/keik/undebuggify) を作ったのでご活用ください。使用例は README.md に書いてあります。


## debug って何

ロガー。個人的に JavaScript でトレースログを埋め込む際は `console.log` ではなく debug を常用している。なぜなら

* ログを埋め込んだモジュールごとに異なる色で出力される
* 前回ログ出力からの経過時間などが出力される
* 環境変数をいじることで出力したいログを絞り込むことができる

など便利な機能があるから。


## なぜ debug を削除する必要があるのか

ウェブブラウザ向けスクリプトをリリースする際、ログ出力コードは削除しておきたい。第一にダサいし、また debug の 4KB のコードをバンドルしたくないから。


## debug じゃなくて console.log 使っているんだけど

[stripify](https://github.com/alanshaw/stripify) をご活用ください。
