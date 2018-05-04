---
title: "Laravel5.6 新規プロジェクト作成手順"
date: 2018-03-26T13:20:38+09:00
categories: ["Laravel"]
draft: false
---

基本に戻ってプロジェクト作成部分を自分がどうやってるか。
世の中には初めてLaravel使ったとか一つのプロジェクトしか見てなくて過剰に盛ってる記事が多すぎる。

## 事前準備

- PHP7.2
- composer
- git
- node.js
- yarn
- vagrant
- PhpStorm

この辺はいちいち説明しなくても用意できてる前提。

## インストール
https://readouble.com/laravel/5.6/ja/installation.html

`laravel/installer`を使うのでまだ入れてない場合はcomposerでグローバルにインストール。
```bash
composer global require "laravel/installer"
```

後は`laravel`コマンドで作成。
```bash
laravel new new-project
cd new-project
```

## ローカルサーバー
https://readouble.com/laravel/5.6/ja/homestead.html

Homesteadをプロジェクトごとにインストール。プロジェクトごとに分けるのが結局一番良かった。
```bash
composer require laravel/homestead --dev
```

```bash
//Mac / Linux
php vendor/bin/homestead make
//Windows
vendor\bin\homestead make
```

`Homestead.yaml`が作られるので環境に合わせて変更。
ipとドメイン程度だけど。

```bash
vagrant up
```
boxのダウンロードが必要な時は時間かかるけど普段はストレスなく使える。
`hosts`の設定もして表示できれば成功。

`php artisan serve`は簡易的な動作確認用なので使わない。

ValetはMacのみなので他の人も関わる場合は選択肢に入れられないけど実際に使ってみても意外と手軽じゃない。

Dockerを開発環境で使う気は全くない。あれは少しのバージョン違いで動かなくなるRailsには必要だけどLaravelではほとんど意味がない。なんとなくでDocker使ってる例をよく見るけどちゃんと考えて本当に必要なのかを判断できてるのは見たことない。
もちろん本番環境で使うかは別の話。

## 設定
https://readouble.com/laravel/5.6/ja/configuration.html
configや.envの設定はお好みで。最初はtimezoneくらいしか変えることはないけど。

## 認証
https://readouble.com/laravel/5.6/ja/authentication.html
デフォルトのviewはwelcomeしかないけど`php artisan make:auth`で作られるファイルまでがデフォルトと思っている。
https://github.com/laravel/framework/tree/5.6/src/Illuminate/Auth/Console/stubs/make

ユーザー登録機能とかを使わないつもりでもここまではやる。

## まとめ
1. laravel new
2. Homestead
3. make:auth

主な工程はこの3つだけ。何回新規に作ったか忘れるくらい作ってるので無駄なことはしない。

これで基本的なアプリはできてるので後はおまけ。

## フロントエンドプリセット
https://readouble.com/laravel/5.6/ja/frontend.html
https://github.com/laravel-frontend-presets
Bootstrap以外を使うなら変更。

なお、この作業は新規プロジェクト作成直後にしかやってはいけない。ごそっとファイル入れ替えてるだけだから開発が進んでからやると吹き飛ばされる。

## フロント周り
```bash
yarn
yarn prod
```

全部設定済みなので必要に応じて変更するだけ。
バージョン付けは使いたいのでwebpack.mix.jsに`.version()`追加してviewを`mix()`に変更はいつもやる。

https://readouble.com/laravel/5.6/ja/frontend.html#writing-vue-components
Vue.js使うなら`resources/assets/js/components`にVueコンポーネントを作って`app.js`で登録すればLaravelのview内でどこでも使える。
Laravel+Vue.jsはここから始めればいい。最初からSPAなんて目指すとたぶん挫折する。

最初からこれだけ用意されてるのになぜか無視してVueコンポーネント使ってない会社を見かけるけどなんでそんな無意味なことをやってるのか全く理解できない。Vueコンポーネント使えば`{{}}`問題もないしScoped CSSも使えるしで一番良い。ベストプラクティスを無視してはいけない。

## composer.json
`barryvdh/laravel-debugbar`とか`barryvdh/laravel-ide-helper`とかいつも使ってるものの追加。
この辺はもはやコピペ。

## .editorconfig
https://github.com/laravel/framework/blob/5.6/.editorconfig
Laravelを参考にするなりして好みで。
これもコピペで同じファイル使い回すだけ。

## StyleCI
GitHubで公開してる場合のみ。
https://styleci.io/

`.styleci.yml`はPSR2にするだけ。

```
preset: psr2
```

## 終わり
全部にURL書いてるように全部ドキュメントに書いてあることでしかない。
検索して変な記事読む前にドキュメントを最初から最後まで全部読むべき。

## 日々の作業
- vagrantの起動／終了
- composer update
- npm script
辺りはPhpStormでやってる。

```bash
alias cu="composer update"
alias yu="yarn upgrade"
```
これを登録してPhpStorm起動するまでもない時は`cu`ですっと更新。
