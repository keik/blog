---
title: slickgrid-colgroup-plugin 作った
date: 2015-12-17 22:11:00
tags: javascript, slickgrid
---

[SlickGrid](https://github.com/mleibman/SlickGrid) のヘッダをグループ化するプラグイン [slickgrid-colgroup-plugin](https://github.com/keik/slickgrid-colgroup-plugin) を作ったので、ギョームシステムの開発とかにどうぞ。MIT。

[デモページ](http://keik.github.io/slickgrid-colgroup-plugin/examples/) にいくつかのサンプルを置いてます。

SlickGrid 自体についても簡単に紹介すると、大量のデータをグリッドで表示できることを特徴としたフレームワークで、その仕組としてはスクロールされるたびに表示領域の DOM 要素だけを動的に作り直すことによって大量データを少ない DOM 要素数で表現している。

プラグイン v1.0.4 における使い方は、SlickGrid オブジェクトにプラグインを登録して

```js
grid.registerPlugin(new Slick.Plugins.ColGroup());
```

カラム定義で `children` プロパティを用いてグループ構造を持ったカラム定義を与えるだけ。

```js
var columns = [
  {id: 'col1', name: 'col 1', children: [
    {id: 'col1-1', name: 'col 1-1', field: 'col1-1'},
    {id: 'col1-2', name: 'col 1-2', field: 'col1-2'}
  ]},
  {id: 'col2', name: 'col 2', children: [
    {id: 'col2-1', name: 'col 2-1', field: 'col2-1'},
    {id: 'col2-2', name: 'col 2-2', children: [
      {id: 'col2-2-1', name: 'col 2-2-1', field: 'col2-2-1'},
      {id: 'col2-2-2', name: 'col 2-2-2', field: 'col2-2-2'}
    ]}
  ]}
];
```
