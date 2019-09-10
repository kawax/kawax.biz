---
title: "Laravel6.0へアップグレード"
date: 2019-09-04T20:18:45+09:00
categories: ["Laravel"]
draft: false
---

## 6.0
バージョニング規約が普通のsemverに変わっただけなのでバージョンアップの手間は5.x時代の0.1と同じ。

5.8までのバージョンは「パラダイム.メジャー.マイナー」
4.2→5.0は大幅に変更されたけどそういう時のためにパラダイムを残していた。
5.xの完成度高くてもうそんな変更はないので一般的なsemverに変えたのでは（予想）

## LTS
前回の5.5期間に判明したことだけど長期サポート対象なのはLaravelフレームワーク本体のみ。Lumenや公式パッケージは含まない。
Socialiteでひと悶着あった。

LTSと言っても安心できないのでいつでもバージョンアップできるようにはしておいたほうが良さそう。

## いつもの見る所
- https://laravel.com/docs/6.0/releases
- https://laravel.com/docs/6.0/upgrade
- https://github.com/laravel/laravel/compare/5.8...master
- https://readouble.com/laravel/6.0/ja/releases.html
- https://readouble.com/laravel/6.0/ja/upgrade.html

今回は`6.0`になったことが一番大きいのでバージョンアップ作業は簡単。
サードパーティパッケージの対応待ち時間のほうが長い。
これ書いてる時点でも待っててほとんどバージョンアップできてないのでなにかあれば後から追記。

## Authorized Resources & viewAny
5.8.xに一度マージされたけどbreaking changesなのですぐ戻された。
「認可」使っててリソースコントローラーのindex()が表示できなくなってたらここをチェック。

## ヘルパー
5.8で非推奨になって対応してるはずだけどサードパーティパッケージには対応漏れがあるかもしれない。
`str_`だけ変更して`starts_with`とかが残ってる例がある。
直してもらうか`laravel/helpers`インストールして対応。

## Input
Inputなんてもう誰も使ってないはずなので非推奨を経ずいきなり削除。

実際には今でも使ってる人がいてびっくり。
もう使ってないと教えても一切話を聞かない。
Laravelはバージョンアップでどんどん変わるから古い知識のままでは使えないってことが理解できてない。

`Input::get()`は`Request::input()`呼んでるだけなので単純に互換のためだけに残されてた。
https://github.com/laravel/framework/blob/7ae3aaaa00f8ab9298104456a59a4154edd5c395/src/Illuminate/Support/Facades/Input.php

## Redis
Redis使うにはPhpRedisをインストール。
https://github.com/phpredis/phpredis

```
pecl install redis
```

php.iniかconf.d/redis.ini

```
extension=redis.so
```

predisは将来非対応になるかも。composerでインストールするだけのpredisのほうが簡単なのでなるべく残して欲しい。

https://laravel.com/docs/6.0/redis

## UI
フロントのファイルが別パッケージに分離。
https://github.com/laravel/ui
`make:auth`も分離されたので今までの新規プロジェクト作ったらまず`make:auth`が変わる。

```
composer require laravel/ui

php artisan ui vue --auth
```

## tests/Bootstrap.php
ファイル追加とphpunit.xml変更。
これ6.0.0リリース後に変更されてる。こういうのが一番困る。
https://github.com/laravel/laravel/pull/5091

## パッケージ作者の対応
Laravel用パッケージではcomposer.jsonに`"illuminate/support": "^5.5",`とか書いてることが多い。
今までは^5.5で5.8まで対応不要でインストールできた。
6.0はインストールできないので`^5.5||^6.0`に変更が必要。

終わり、ではない。
リリースサイクルは変わらないので半年後には7.0が出る。
また`^5.5||^6.0||^7.0`に変更。
これを半年毎に繰り返せ…？

付き合ってられないならバージョン指定消して`*`にすれば何もしなくて良くなる。

## その他
ローカルで動かさずバージョンアップ作業だけしてたようなプロジェクトは6.0後に一度`php artisan view:clear`したほうがいいかも。本番でも。
viewのキャッシュに削除されたarray_ヘルパーが残っててエラーになる。キャッシュ削除すれば直る。