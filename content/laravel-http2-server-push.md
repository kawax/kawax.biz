---
title: "Laravel で HTTP/2 Server Push"
date: 2018-03-14T22:05:40+09:00
categories: ["Laravel"]
draft: false
---

2018年2月リリースのnginx1.13.9がHTTP/2 Server Pushに対応したことで必要な環境が揃って新時代の幕開けの予感。
Apacheや有料版nginxでは結構前から対応していたけどノーマルなnginxが対応したので普及しやすくなる。
最近使ってるUbuntuではまだ1.13.6なので今の内に対応準備。

## Laravel
これ使えば良さそう。  
https://github.com/tomschlick/laravel-http2-server-push

```bash
composer require tomschlick/laravel-http2-server-push
```

（v0.1.6時点ではLaravel5.6でインストールできないので`dev-master`で指定。）

Auto-discovery対応してるのでServiceProvider追加は不要。

`app/Http/Kernel.php`に追加

```php
protected $middleware = [
    \TomSchlick\ServerPush\Http2ServerPushMiddleware::class,
];
```

`vendor:publish`で`config/server-push.php`がコピーされるのでLaravel Mix用に変更。デフォルトではElixir用になっている。
```bash
php artisan vendor:publish
```

Mixのcss/jsは自動で追加される。他にも追加したいならserver-push.phpで設定するか、動的に設定したいなら`pushImage()` `pushScript()` `pushStyle()`のヘルパーを使う。

Mixで`version()`を使ってると少しだけ上手く動かない。
Mixの`app.js?id=aaaa`に対応できてないので自分でパッチ当てて使う。
https://github.com/kawax/laravel-http2-server-push/commit/71450921b562e57c714c1b0ce89e025cb37fe8ae

## Netlify
ついでに他も調査。  
https://www.netlify.com/blog/2017/07/18/http/2-server-push-on-netlify/

NetlifyはHugoなら`/static/_headers`に書けばいい。

## WordPress
プラグインで対応できるけどまだどれがいいとかはない。  
ここで実験していく。https://wp-update.info/

## nginxの設定
その後Ubuntuで使えるようになったのは1.13.12。
nginx側の設定としては以下を追加するだけ。

```
http2_push_preload on;
```

実際に対応してみるとすでに十分速いのであまり変わらなかった。Server Pushするファイルを増やせば変わるかも。
