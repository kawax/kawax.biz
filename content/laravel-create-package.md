---
title: "Laravel: APIを使う独自パッケージの作り方"
date: 2017-12-26T21:12:24+09:00
categories: ["Laravel"]
draft: false
---

openBDを例に。https://openbd.jp/

## 基本
- APIの情報をよく調べる。openBDの場合登録は不要だけど普通は必要。
- Laravel専用でしか作れないものでない限り普通のcomposerパッケージとして他でも使えるように作る。Laravel対応は最終段階で。

## 新規プロジェクトを作る
パッケージだけ作っても動作確認できないのでまずは普通にLaravelプロジェクトを作る。

```
laravel new laravel-openbd-project
cd larave-openbd-project
```

今回はDB不要なのでローカルサーバーは artisan serve を使う。

```
php artisan serve
```

## 仮のパッケージ置き場
プロジェクト内に `lib/laravel-openbd/` を作りパッケージ用のファイルはここに作っていく。

名前空間を決める。 `Revolution\OpenBD`

composer.json の `psr-4` に追加。

```
"autoload": {
        "classmap": [
            "database/seeds",
            "database/factories"
        ],
        "psr-4": {
            "App\\": "app/",
            "Revolution\\OpenBD\\": "lib/laravel-openbd/src/"
        }
    },
```

これでこのプロジェクト内では `Revolution\OpenBD` は `lib/laravel-openbd/src/` から読み込まれる。ある程度完成するまではこの状態で開発。

## Client
APIを使う場合HTTPクライアントとしてGuzzleを使う。  
https://github.com/guzzle/guzzle

composerに追加。

```
composer require guzzlehttp/guzzle
```

Guzzleをプロパティで持った `OpenBD.php` を作る。今回のパッケージはこれがメイン部分。

Googleなんかも同じ作り方してるので真似する。
https://github.com/google/google-api-php-client

```php
<?php

namespace Revolution\OpenBD;

use GuzzleHttp\Client;
use GuzzleHttp\ClientInterface;

class OpenBD
{
    /**
     * @var ClientInterface
     */
    protected $http;

}
```

## __construct
今回は不要だけど設定情報などが必要なら `__construct` で渡す。Laravelならconfig経由で。

```php
/**
 * OpenBD constructor.
 */
public function __construct()
{
    $this->http = new Client();
}
```

ついでに Guzzle の setter/getter。`ClientInterface` で指定しておくとGuzzle以外でも使えるはず。
`setHttpClient` の `return $this` は `setHttpClient()->` とメソッドチェインできるようにするため。

```
/**
 * @param ClientInterface $http
 *
 * @return $this
 */
public function setHttpClient(ClientInterface $http)
{
    $this->http = $http;

    return $this;
}

/**
 * @return ClientInterface
 */
public function getHttpClient(): ClientInterface
{
    return $this->http;
}
```

## endpoint
将来変更された時のためにユーザーが変更できるようにしておくべき。

```
/**
 * @var string
 */
protected $endpoint = 'https://api.openbd.jp/v1/';

/**
 * @param string $endpoint
 *
 * @return $this
 */
public function endpoint(string $endpoint)
{
    $this->endpoint = $endpoint;

    return $this;
}
```

## API
openBDのAPIは少ないので全部メソッドにする。多い場合は全部作るわけにはいかないので `call()` みたいなメソッドを作ってユーザーが指定する形にする。

```
get()
coverage()
schema()
```

## 動作テスト
今回は簡易的なものなので routes/web.php に書く。

```
use Revolution\OpenBD\OpenBD;

Route::get('coverage', function () {
    dd((new OpenBD)->coverage());
});

Route::get('schema', function () {
    dd((new OpenBD)->schema());
});

Route::get('get/{isbn}', function ($isbn) {
    dd((new OpenBD)->get(explode(',', $isbn)));
});
```

問題なければ重要な部分は完成。

## Interface
ここで `OpenBD.php` から Interface を作る。PhpStorm など IDE なら簡単にできるはず。
PHP7なら型情報もあるし `OpenBDInterface.php` を見るだけでどんなメソッドを持つか分かる。

## パッケージに分離
`lib/laravel-openbd/` を別の場所にコピー。

`composer init` でパッケージ用のcomposer.jsonを作る。大抵は他で作ったcomposer.jsonをコピーしてきて書き換えることになる。

今からならPHPは7.0以上でいい。基本的にはLaravelの要求バージョンと合わせる。

```
"php": ">=7.0.0",
```

autoload
```
"autoload": {
    "psr-4": {
        "Revolution\\OpenBD\\": "src/"
    }
},
```

.gitignore。パッケージには `composer.lock` は含めない。
```
/vendor
composer.lock
```

## ServiceProvider と Facade を作る
ここも他からコピーしてきて書き換えることがほとんどなので1回覚えれば理解できる。

Laravel5.5からの Package auto discover 用にcomposer.jsonに追加。

```
"extra": {
    "laravel": {
        "providers": [
            "Revolution\\OpenBD\\Providers\\OpenBDServiceProvider"
        ],
        "aliases": {
            "OpenBD": "Revolution\\OpenBD\\Facades\\OpenBD"
        }
    }
}
```

## テスト
composer.json。mockeryは使ってないのでなくてもいい。
```
"require-dev": {
    "phpunit/phpunit": "6.*",
    "mockery/mockery": "^1.0"
},
```

Guzzleのテストは `MockHandler` を使えばモックできる。

## 一旦公開
GitHub、Packagistと諸々設定してcomposerでインストールできるようにする。

https://packagist.org/packages/revolution/laravel-openbd

## プロジェクトに戻る
composer.jsonのpsr-4の部分は削除。`lib` 以下も削除していい。
```
"Revolution\\OpenBD\\": "lib/laravel-openbd/src/"
```

composer.jsonに追加。この時点ではまだバージョンはないのでdev-master。すぐには反映されないかもしれないのでインストールできるまで待つ。

```
"revolution/laravel-openbd": "dev-master"
```

`composer update` が問題なく完了するようになったら再度動作テスト。

Facadeを使った場合。

```
Route::get('coverage', function () {
    dd(\OpenBD::coverage());
});

Route::get('schema', function () {
    dd(\OpenBD::schema());
});

Route::get('get/{isbn}', function ($isbn) {
    dd(\OpenBD::get(explode(',', $isbn)));
});
```

composer からインストールした状態で問題なければ完成。

## 仕上げ
README.md を書く。OpenBDは日本人向けなので日本語でもいいけど基本的には簡単でもいいので英語が推奨。

`Macroable` もとりあえずで対応させておく。

gitのtagがバージョンになる。未完成と思えば `0.0.1` にしてもいいしもうそんなに変更することがなさそうなら最初から `1.0.0` でもいい。openBDはシンプルだったけど普通はこんなことないのでもっと作り込む必要がある。

プロジェクト側のcomposer.jsonのバージョンを書き換えてupdateして問題なければ完了。

https://github.com/kawax/laravel-openbd  
https://github.com/kawax/laravel-openbd-project
