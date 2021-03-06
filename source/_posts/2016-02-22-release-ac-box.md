---
title: オートコンプリートライブラリを作った話と UI 実装のテクニックについて
date: 2016-02-22 22:49:25
tags:
- javascript
- ac-box
- autocomplete
- combobox
- flux
---

シンプルなオートコンプリートコンボボックス UI ライブラリ [ac-box](https://github.com/keik/ac-box) を作った。[デモはこちら](http://keik.info/ac-box/examples/)。

## 特徴

* 他の大きなライブラリに依存しない。スタンドアロン版ミニファイ済み 9KB。
* UI の位置は自動で fix される。ボーダや背景などの装飾部分は独自に定義する。


## 使い方

[README](https://github.com/keik/ac-box#readme) を参照してください。


## Typeahead とか jQuery UI とかあるけど？

jQuery に依存したくない。


## 〜する機能はないの？

[issue](https://github.com/keik/ac-box/issues) にもらえれば対応できるかもしれないです。


# UI 実装のテクニック

オートコンプリートは、シンプルな機能ながらも微妙に細かなインタラクションがあるため、UI 実装の練習になる。このライブラリでも使っている UI 実装のテクニックをいくつかメモした。興味のある方は参考にしてください。


## Flux

[Flux](http://facebook.github.io/flux/docs/overview.html#content) アーキテクチャを実装している。[ミニマムな Event Emitter はこんな感じ](https://github.com/keik/ac-box/blob/master/src/event-emitter.js)。このライブラリだとストアの操作がほとんどないので、あまり意味はないけれど。

Flux アーキテクチャは、MVC アーキテクチャの一つの例でしかない。ところが、最近は MVC アーキテクチャの派生系が次々に登場してきて、もうこれ何が MVC だかわかんねえなっていう状況の中、Observer Pattern + Command Pattern による MVC アーキテクチャに単一データフローという制約を加えたもの改めて Flux という名前で再定義した。そうすることでたびたび発生していた MVC カオスに巻き込まれるのを回避して、初学者にも混乱なく分かりやすく使い始められるという印象を受ける。


## View ステートとレンダラーによる描画

UI の状態制御には、次のような、View ステートとレンダラーを使った実装方法がある。

1. イベントハンドラ内で、次のような View ステートのプロパティを更新する
```javascript
this.state = {
  // オートコンプリートメニュー開閉状態。input にフォーカスあたったら true にしたりする
  isOpen: null,
  // input に設定する値。ESC でメニュー選択を中断したりしたいので、DOM 要素のプロパティ外にステートとして保持する
  value: null,
  // オートコンプリートメニュー内でフォーカスがあたっている場所を表す数字。UP とか DOWN でインクリ / デクリメントする
  focusedIndex: null
}
```
2. その後レンダリング関数をコール。レンダリング関数は View ステートを参照して DOM 要素を更新する

この方式の場合、レンダリング関数が割とクソ実装な見栄えになってしまうことがある。それでも、そこそこ複雑なインタラクションのある UI を制御するにはこの方法がいい。

対比として、イベントハンドラ内で直接 DOM 要素を更新する実装を考えた場合、連続的に発生するイベントに対して表示状態の一貫性が保てずに破綻しがちになる。例えば、`click` イベントでメニューを選択するというイベントハンドラを実装する際、気持ちの上では `click` のみ Listen したい。だが実際には `mousedown` や `mouseup` や `blur` や `focus` などのイベントが連続的に発生する。これらのイベントハンドラと整合させつつ描画を更新する場合に View ステートというクッションを挟むことで、制御がしやすくなる。オプションで動作にバリエーションをもたせるような場合にも拡張しやすい。

そして View ステートから DOM 要素の更新を最小パッチで自動的に行うのが、我らがイケメン [React](https://facebook.github.io/react/) なのでライブラリではなくアプリを作るなら React 使うといいと思う。他に [RxJS](https://github.com/Reactive-Extensions/RxJS) も気になるが、まだ使ったことがない。


## 画像を使わない

メニュー展開状態を示すキャレットアイコンなどは CSS の `border` プロパティで作り、画像は使っていない。外部アセットファイルを増やしたくないから。Bootstrap のキャレットもこうやって作られているし、割と定番なテクニック。


## トレースログを埋め込む

UI 実装は、ユーザ操作とのインタラクションによってコールされる関数の順序性などを確認しながら行いたいもの。それにパフォーマンスも気になる。そこで、`console.log` ではなく [debug](https://github.com/visionmedia/debug) を使って各関数の頭にトレースログを埋め込むというのをオススメ。簡易なパフォーマンス計測やトレースログのフィルタやログの色分けなどができて便利。

そしてリリースビルドには debug をバンドルしたくないので、[undebuggify](https://github.com/keik/undebuggify) で debug 関連コードを除去するトランスフォームをすると具合がいい。
