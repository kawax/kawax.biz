---
title: "env()はconfigファイル内でしか使えない"
date: 2019-04-26T22:26:31+09:00
categories: ["Laravel"]
draft: false
---

なぜか何度説明しても納得しない人が多い。

## ドキュメント
日本語訳だと若干分かりにくい。

> Note: 開発期間中にconfig:cacheコマンドを実行する場合は、設定ファイルの中で必ずenv関数だけを使用してください。設定ファイルがキャッシュされると、.envファイルはロードされなくなり、env関数の呼び出しは全てnullを返します。

https://readouble.com/laravel/5.8/ja/helpers.html#method-env

> If you execute the config:cache command during your deployment process, you should be sure that you are only calling the env function from within your configuration files. Once the configuration has been cached, the .env file will not be loaded and all calls to the env function will return null.

https://laravel.com/docs/5.8/helpers#method-env

deploymentが「開発」になってるのでそもそも誤訳か。

本来はデプロイ、つまり本番環境で`php artisan config:cache`コマンドを使って最適化するならenv()はconfigファイルでしか使えない。`config:cache`後はenv()はnullしか返さないから。開発時でも`config:cache`すれば同じだけど。

`config:cache`しないなら使ってもいいけどドキュメントではすべきとなっている。
https://readouble.com/laravel/5.8/ja/deployment.html#optimizing-configuration-loading
https://laravel.com/docs/5.8/deployment#optimizing-configuration-loading
ここでもまたenv()に関する注意。知らずに`config:cache`したら100%アプリは壊れるのでしつこく注意。

良いとか悪いとか好みとか一切関係なくこれがLaravelのルールなので従う以外にない。

## 実際に動かして試す

```php
Route::get('env', function () {
    dd(env('APP_ENV'));
});
```

cacheしてない→`local`

cacheしている→`null`

## フレームワークの中身を確認
キャッシュファイルが存在すればここで読み込んでるのは分かる。

```php
// First we will see if we have a cache configuration file. If we do, we'll load
// the configuration items from that file so that it is very quick. Otherwise
// we will need to spin through every configuration file and load them all.
if (file_exists($cached = $app->getCachedConfigPath())) {
    $items = require $cached;
    $loadedFromCache = true;
}
```
https://github.com/laravel/framework/blob/9fb420cc29a7dd5de5051f09c523ffc3ea01b969/src/Illuminate/Foundation/Bootstrap/LoadConfiguration.php#L27

env()でnullになるのはどこなのかは少し探しただけでは分からなかった。

ここですぐreturnされて何も読み込まれてないからかな。

```php
if ($app->configurationIsCached()) {
    return;
}
```
https://github.com/laravel/framework/blob/9fb420cc29a7dd5de5051f09c523ffc3ea01b969/src/Illuminate/Foundation/Bootstrap/LoadEnvironmentVariables.php

この先はLaravel外パッケージの調査が必要。

## 他のパッケージのenv()と競合したら
滅多にないけどenv()はグローバル関数なので他のパッケージにenv()が含まれていたら競合して壊れることがある。
例えば。
https://github.com/cakephp/core/blob/f9d9823439e3c82fdb3f69d2c9210bab67e721af/functions.php#L189

Laravelでcakeのパッケージなんてインストールしないと思うだろうけど
自分でインストールしなくても何かのパッケージの依存先でインストールされるかもしれない。
知らない内に競合してると気付かないので怖い。

対策はインストールされないようにcomposer.jsonのconflictで指定するしかない。これが必須なパッケージだったらインストールしないせいで壊れるのでどうしようもない。

```json
"conflict": {
    "cakephp/core": "*"
},
```

この対策としてLaravel 5.9で`Illuminate\Support\Env`が追加されるはずなのでグローバルなenv()を使わない回避手段ができる。
実際使うとなると全configファイルを修正なので面倒だけど。かなり特殊な用途なので仕方ない。

```php
use Illuminate\Support\Env;

return [
  'env' => Env::get('APP_ENV');
]
```
