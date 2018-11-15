---
title: "Laravel 5.4へアップグレードと未対応ライブラリの一時的対応"
date: 2018-10-09T02:29:47+09:00
categories: ["Laravel"]
draft: true
---

（Qiita/Kobitoからサルベージ。2017/01/29）

## 準備
Laravel本体の5.3->5.4はアップグレードガイド見ながら書き換えればそんなに作業量はなかった。
https://laravel.com/docs/5.4/upgrade
プロジェクト内の細かい変更点はGitHubで。
https://github.com/laravel/laravel/compare/5.3...master

## tests
つらつらと書き換えてたけど最後のtests部分が何気に一番変わっていた。
https://laravel.com/docs/5.4/testing
https://laravel.com/docs/5.4/http-tests

```php
   $this->visit('/')
        ->see('Laravel');
```
この書き方はLaravel Duskに移って消滅？

`seeInDatabase`は`assertDatabaseHas`に。
https://laravel.com/docs/5.4/database-testing

これを使えば5.3スタイルで引き続き使える。
https://github.com/laravel/browser-kit-testing
共存も可能なので移行が大変なら5.3部分はそのまま残して新しいテストから5.4スタイルにするのが良さそう。

## 未対応ライブラリ
Laravel本体は簡単でも他のライブラリが対応できてないことは多い。
簡単に直せるなら自力でどうにかできる。
例えばthujohn/twitterなら

1. https://github.com/thujohn/twitter をFork
2. patchブランチを作る
3. https://github.com/thujohn/twitter/pull/170/files と同じように修正
4. composer.jsonを変更

自分のurlで。

```json
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/*/twitter"
        }
    ],
```

```
        "thujohn/twitter": "dev-patch",
```

この辺の機能
https://getcomposer.org/doc/05-repositories.md#loading-a-package-from-a-vcs-repository

thujohn/twitterが対応したら元に戻す。

"5.3.*"を指定してるからインストールできない程度ならこれで対応できるけど全く動かないものもあるので諦めて待つか使わないようにするしかない。
