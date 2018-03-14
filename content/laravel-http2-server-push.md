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
