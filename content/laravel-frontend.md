---
title: "Laravelとフロントエンド（初級）"
date: 2019-02-17T20:14:04+09:00
categories: ["Laravel"]
draft: false
---

Laravel5.7で新規プロジェクトを作った直後の状態で見るファイル。

## package.json
composer.jsonと同じ役割。
Laravel使うならcomposer使うのが当然なようにJS/CSSに対してもnpmを使う。

## webpack.mix.js
Laravel Mix用のファイル。
`.version()`を追加する程度で普通に使う範囲なら触らなくていい。

以前のLaravel Elixirはgulp+webpackだったけどMixはwebpackのみのラッパー。

## resources/js/bootstrap.js
jQueryなどを読み込んでるファイル。起動処理を書く部分。
npmで追加したライブラリもここで読み込めばいい。
読み込む方法は`require`でも`import`でもどっちでもいい。
webpackが上手いことやってくれることであり詳細は気にしなくていい。

ダウンロードしてきたファイルをpublicに置いて<script>で読み込むなんて使い方はしない。

## resources/js/app.js
アプリのメイン処理を書く部分。
Vue.js使うならVueコンポーネントの登録しかすることないけど
jQueryをここに直接書いてもいい。
別ファイルに分けて`require`で読み込んでもいい。

viewに直接書くような使い方はしない。

## resources/sass/app.scss
CSS用のファイル。

## ビルド
ここ数年で変わったのはJS/CSSもビルドして使うものになったということ。
常識レベルでの大変革なのでダウンロードしてきたファイルを<script>で読み込んで使うという常識を全部投げ捨てないといけない。

Laravel Mixならビルド用のコマンドもpackage.jsonに用意されてるので以下でビルド。

```
# 開発時
npm run dev
# 本番用
npm run prod
```

ビルド後のファイルがpublicに生成される。読み込むのはこのファイル。
それ以外のJS/CSSファイルは直接publicに置かない。

CSSはhead

```
<link rel="stylesheet" href="{{ mix('css/app.css') }}">
```

JSは</body>前。

```
<script src="{{ mix('/js/app.js') }}"></script>
```

これによりJS/CSSを変更する時に触るのはresources以下のみ、view側は触らなくていい、となる。

## 以上
Vue.jsとかSPAとか言う前にこのくらいの基礎中の基礎が分かってないとどうしようもない。

npmの使い方とかSASSの使い方とか現代JSの書き方とかそんな段階の話はLaravel使うなら知ってて当然の前提。
