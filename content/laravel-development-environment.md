---
title: "段階別Laravelの開発環境"
date: 2019-01-21T15:20:12+09:00
categories: ["Laravel"]
draft: false
---

## レベル1
```
php artisan serve
```
https://laravel.com/docs/master/installation
https://readouble.com/laravel/5.7/ja/installation.html

Laravelの新規プロジェクト作成ができてる=PHPとcomposerが動くなら確実に動くので最初はこれだけでいい。

**データベースを使わない** 範囲ならこれで十分。

初心者と極めすぎてartisanコマンドしか使わなくなった人向け。

## レベル1.5
Valet
https://laravel.com/docs/master/valet
https://readouble.com/laravel/5.7/ja/valet.html

Macのみなので省略するけどウェブサーバー+データベースなので`php artisan serve`の次の段階としては一番ちょうどいい。どっちにしろすぐに足りなくなるので次へ。

## レベル2
Homestead
https://laravel.com/docs/master/homestead
https://readouble.com/laravel/5.7/ja/homestead.html

Laravelの全機能使うための環境が全部揃ってるのでデータベースが必要になって以降は基本的にはHomesteadだけ使えばいい。

重要な点はHomesteadまではLaravelの公式サポート内ということ。
初心者が非公式なものに手を出してはいけない。

Homestead使ってる段階のうちにいくつもプロジェクト作ったり運用したりして全機能試す。
cron、キュー、ブロードキャスト辺りまで全部。
Laravelのこの機能のためにサーバー側に何が必要かを理解する。

Homestead内に何がインストールされてるかはこれで分かる。
https://github.com/laravel/settler

## レベル3
DockerだけどLaradockは使わない。Laravelと関係ないものまで入れすぎてて過剰。複数プロジェクトで使いにくい。
ここまでの過程を経ずにいきなりLaradock勧めてる人の話は無視したほうがいい。初心者向けではない。

Docker使いたい段階になったらどうするかというと自分で一から作る。
Homesteadの経験があれば必要なものは分かるのでnginx, php, mysql, redis辺りの公式イメージからdocker-compose.ymlを自分で作れるようになる。
作れないならDocker使うのは早いのでHomesteadに戻ったほうがいい。

日本語での「DockerでLaravel開発環境作ってみた記事」はPHPとMySQL程度しか使ってなくて全く役に立たないので検索しても無駄。
cronもキューも使えないLaravel開発環境に価値はない。

Laravel公式にDocker扱う気はなさそう。
https://twitter.com/taylorotwell/status/849739001812725760
Docker使いたい人ならもう自力でどうにかできるだろう。
はっきり言ってLaravelで何か作るのは誰にでもできる簡単なこと。
Dockerやインフラの深淵のほうが遥かに難しい。
公式にHomesteadを用意してるのは難しい部分を省略してすぐに開発始められるようにするためなのでそれが必要なくなった段階の人へのサポートはない。


## XAMPPは使ってはいけない
Laravelの開発環境としては何もかも足りない。
どこかの本はXAMPP使ってるらしいけどそんな本は投げ捨てたほうがいい。
XAMPP使ってる時点でLaravelのことが何も分かってない。

Web職人のためのPHPフレームワーク「Laravel」をマスターしよう！～HelloWorldを表示させる (1/3)：CodeZine（コードジン）
https://codezine.jp/article/detail/11231

この記事読んでひどすぎると理解できるようになろう。（今は2ページ目以降読めないけど）


検索してみれば分かるけどLaravelでXAMPP使ってるのは日本人ばかり。本の影響。
（stack overflowレベルまで見れば海外にも少人数ながらいるけど）
XAMPPで始めた人のブログのその後を見てもLaravelをまともに使えるようにはなってない。
