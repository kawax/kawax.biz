---
title: "Laravel5.8の準備：ヘルパー"
date: 2019-02-23T21:06:32+09:00
categories: ["Laravel"]
draft: false
---

Laravel5.8では一部のヘルパー(`array_*`と`str_*`と`*_case`等)が非推奨になる。たぶん5.9で削除。
すでに大量に使ってるなら修正せずにこのヘルパーパッケージを使ってそのまま使い続ければいい。
https://github.com/laravel/helpers
自分のプロジェクト内は修正しても結局外部のパッケージが使ってて必要になると予想してる。
パッケージ作ってる人は対応しよう。

## Bladeでは
この提案した人Blade書いてないのかよってほどにBladeでは不便になる。
https://github.com/laravel/framework/pull/26898

ヘルパーならこれだけなのに

```php
{{ str_limit($content, 140) }}
```

フルで書くとこれ…。

```php
{{ \Illuminate\Support\Str::limit($content, 140) }}
```

ひどいけど良い解決方法があった。

config/app.phpの`aliases`に追加。

```php
'aliases' => [
    ...

    'Str'          => \Illuminate\Support\Str::class,
    'Arr'          => \Illuminate\Support\Arr::class,
],
```

こう書けるので短くなった。

```php
{{ Str::limit($content, 140) }}
```

ヘルパー使わないならこれ。
