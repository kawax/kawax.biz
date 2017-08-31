---
title: "Laravelから単体で使えそうなコンポーネント"
date: 2017-08-30T17:07:11+09:00
categories: ["Laravel"]
draft: false
---

<!--more-->

## illuminate/support
https://github.com/illuminate/support

```
composer require illuminate/support
```

これでこの辺りのヘルパーが使えるようになる。
https://readouble.com/laravel/5.4/ja/helpers.html

Laravel外では意味のないものもあるけど配列や文字列辺りは使えるはず。
後はCollectionもあるので便利な機能が使える。
https://readouble.com/laravel/5.4/ja/collections.html

PHPライブラリ作ってる時にLaravelのあれ使いたいなぁと思うことがよくある。
そういう時はこれ使えば大体解決する。

## Laravel Mix
LaravelでもPHPでもないけどwebpackの軽いラッパーとしてJSだけのプロジェクトでも使いたいくらい。
https://github.com/JeffreyWay/laravel-mix
少し工夫すればRailsとかでも使えるはず。

## Eloquent
これは一番需要ありそうなので情報はいっぱいあるはず。
READMEに使い方書いてあるのでいいだろう。
https://github.com/illuminate/database
これを単体で使うならそもそもLaravel使うけど。

## Blade
WordPressのテーマでBlade対応してる例
https://github.com/roots/sage
