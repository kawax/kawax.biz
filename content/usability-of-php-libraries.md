---
title: "PHP用ライブラリの使いやすさ"
date: 2020-10-26T17:22:46+09:00
categories: ["Laravel"]
draft: false
---

今のPHP用ライブラリの使いやすさ評価には「名前空間の切り方」や「IDEでの補完しやすさ」まで含まれる。

Laravelはこの辺りかなり意識的。
例えばJetstreamの`HasProfilePhoto`や`HasTeams`トレイト。
`Laravel\Jetstream\HasProfilePhoto`みたいにsrc直下に置かれている。
https://github.com/laravel/jetstream/tree/1.x/src
以前のLaravelならTraitsにまとめてる。
https://github.com/laravel/framework/tree/8.x/src/Illuminate/Support/Traits

なぜこうなっているかというと`HasProfilePhoto`が使われるのはUserモデルだから。
```php
<?php

namespace App\Models;

use Laravel\Jetstream\HasProfilePhoto;

class User extends Authenticatable
{
    use HasProfilePhoto;
```
LaravelのJetstreamのHasProfilePhoto。必要最小限。ここにTraitsも足すと余計な情報。
実際にユーザーが使う場面まで考えてる。

Laravel開発者の考えなので正解は分からないけど。

## Amazonの例
Product Advertising API用のSDKがここにzipで置いてある。
https://webservices.amazon.com/paapi5/documentation/quick-start/using-sdk.html
ダウンロードしたREADMEには`composer require amzn/paapi5-php-sdk`なんて書いてあるけどpackagistに登録してないしGitHubにもないのでcomposerでインストールするAmazon公式な方法は一切ない。

zipでダウンロードして、プロジェクト内に直接入れてcomposer.jsonのautoloadを調整すれば使える。

いざ使おうとして見るのはこの名前空間。
```php
use Amazon\ProductAdvertisingAPI\v1\com\amazon\paapi5\v1\api\DefaultApi;
```
Laravelの例と比べると異常さがよく分かるだろう。
なんとなくJavaしか知らない人が作ったような雰囲気。
実際はSwaggerから自動生成しただけだからこんなことになってるっぽい…。

## 使いにくい例
こういう名前空間だとする。FooクラスとFooディレクトリとBarクラスが存在する。
```php
use Sample\Foo;
use Sample\Foo\Bar;
```
先にFooクラスだけインポート。
```php
use Sample\Foo;

$foo = new Foo();
```
ここからBarクラスを自動補完を期待して入力しようとすると
```php
use Sample\Foo;

$foo = new Foo();
$bar = new Foo\Bar();
```
PhpStormだとBarを入力しようとしても`Foo\Bar`の形になる。

期待する↓の形にはならない。
```php
use Sample\Foo;
use Sample\Foo\Bar;

$foo = new Foo();
$bar = new Bar();
```

`Foo\Bar`でも正しく動作する。
Foo以下に複数のクラスがあるならこれが便利な時もあるけど基本的には意図してないIDEの挙動。
```php
use Sample\Foo;

$foo = new Foo();
$bar = new Foo\Bar();
$baz = new Foo\Baz();
```

これが使いにくい名前空間の切り方の例。
ライブラリを作ってる側からはキレイに見えるけど使う側からは不便。
