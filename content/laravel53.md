---
title: "Laravel5.3 アップグレード"
date: 2018-10-09T02:28:40+09:00
categories: ["Laravel"]
draft: false
---

（Qiita/Kobitoからサルベージ。2016/08/27）

リリース前から触ってて良かったなぁと思うくらい意外と大変。
複雑なことしてないプロジェクトなら簡単だけど。

基本的には公式のドキュメント見ればいい。
https://laravel.com/docs/5.3/upgrade

## 事前準備
5.3のプロジェクトをzipでダウンロードしておく。ファイルごと入れ替えたほうが早い所も多いので。
https://github.com/laravel/laravel

## routes
プロジェクトルートにコピー。
app/Http/routes.phpの中身をroutes/web.phpにコピペ。
api用のルーティングはapi.phpへ。
api.phpやconsole.phpを使わないなら不要な部分は削除しておく。

app/Http/routes.phpは削除。

## app/Providers
各ファイルを入れ替え、もしくは中身をよく見て書き換え。
AuthServiceProvider.php
BroadcastServiceProvider.php
EventServiceProvider.php
RouteServiceProvider.php

Routeが重要。

BroadcastServiceProvider.phpはconfig/app.phpでは読み込んでないので使わないなら不要。

ここを変えないまま5.3にするとたぶん確実にエラーになるのでアップグレードガイドでも最初に書いてある。

## Arr
前に書いたfirst()などの引数の順番。これも使ってると確実に互換に問題出るので2番目。

## config/app.php
3箇所追加。

```php
    'name' => 'My Application',
```

```php
        Illuminate\Notifications\NotificationServiceProvider::class,
```

```php
        'Notification' => Illuminate\Support\Facades\Notification::class,

```
Mailの下。

## User
Notifiable追加。

```php
use Illuminate\Notifications\Notifiable;

...

    use Notifiable;

```

## app/Http/Controllers/Controller.php
AuthorizesResourcesを削除。

## app/Http/Controllers/Auth
全部変わってるので全部入れ替え。

## app/Http/Kernel.php
変更してないならそのまま入れ替え。
「モデル結合ルート」が動いてなくて変だと思ったらMiddlewareで処理するようになってたのでここを忘れると危険かも。

testでMiddleware無効にしてると不便なのでまた変わるかもしれないけど。

## app/Http/Middleware/Authenticate.php
削除。

## app/Http/Requests
使ってないならフォルダごと削除。
使ってるならRequest.phpを使わないように変更。
`php artisan make:request TestRequest`してみれば少し書き換えるだけと分かる。

## Events,Jobs,Listeners,Policies
この辺も使ってないなら削除。
使ってるならRequestと同じように修正していくしかない。

## Pagination
bootstrap3使うなら何もしなくていいはず。

## config/queue.php
`expire`と`ttr`を`retry_after`に書き換え。

sqsはprefixとqueueに分かれてるので使ってるなら変更。

## キュー
queue:listenがなくなってworkだけになった？
`php artisan queue:work`がデーモンとして起動
`php artisan queue:work --once`は一度だけ。

## composer.json
5.3用にバージョン上げる。

```
    "laravel/framework": "5.3.*",
```

laravelcollective使ってるなら今はまだ@dev付けたほうがいいかも。

```
    "laravelcollective/html": "5.3.*@dev",
```

`composer update`

## 終わり
他はpackage.jsonとかだけど大体分かるだろう。

とりあえずこのくらい変更すれば5.2から5.3へはできるはず…。
後は他のパッケージが5.3に非対応だったりすると待つしかなくなる。
幸い一番大きいプロジェクトは早めに準備してたのですぐ5.3にできそう。
