---
title: "Laravel Socialite 独自ドライバの作り方"
date: 2018-05-10T10:43:45+09:00
categories: ["Laravel"]
draft: false
---

（Qiita/Kobitoからサルベージ）



[Laravel SocialiteをベースにしてLaravelにLINE loginを実装するための手順](https://yukiyuriweb.com/implement-line-login-in-laravel-with-laravel-socialite/)
何気なく見てたら思いっきり間違ってること書いてた…。`vendor/` 以下は composer update で書き換わるから直接触ってはいけない。

http://qiita.com/LowSE01/items/d07562098ca416076b2c
これも微妙に違う。
`Laravel\Socialite\SocialiteServiceProvider` ごと入れ替えるとかさらに追加したい時は？

http://qiita.com/mayutan/items/9ff945f56718f5cb42cf
これが正しいけど惜しい。自分でも間違いに気付いたのは最近だけど。これも結局 SocialiteManager を LineManager に入れ替えてる。自分のアプリ内だけならこれで問題ないので探して見つかるのは大体このやり方。

https://socialiteproviders.github.io/
なお、ここにいっぱいあるけどイベント使ってるせいでいまいち。ここのは使ったことない。

## 参考にするもの
もちろん公式。
https://github.com/laravel/socialite

各 Provider のどれかをコピーして使うことになる。

## バージョン
- Laravel 5.5
- Socialite 3.0

## 作るファイル
- ServiceProvider
- Provider

どうせ細かい違いしかないので中身はどこかからコピーしてきて書き換えればいい。と言っても微妙に間違ってるのも多いので ServiceProvider だけ下に書く。Provider は他と同じ。

パッケージにするなら composer.json も。バージョン指定は`*`でいい。

```json
  "require": {
    "laravel/socialite": "*"
  },
```

```json
  "autoload": {
    "psr-4": {
      "My\\Socialite\\Line\\": "src"
    }
  },
```

Package discover 用に追加。

```json
  "extra": {
    "laravel": {
      "providers": [
        "My\\Socialite\\Line\\LineServiceProvider"
      ]
    }
  }
```

こういう構成。

- src/LineServiceProvider.php
- src/LineProvider.php
- composer.json

## ServiceProvider

```php
namespace My\Socialite\Line;

use Laravel\Socialite\SocialiteServiceProvider;
use Laravel\Socialite\Contracts\Factory;

use Laravel\Socialite\Facades\Socialite;

class LineServiceProvider extends SocialiteServiceProvider
{
    /**
     * Indicates if loading of the provider is deferred.
     *
     * @var bool
     */
    protected $defer = false;

    /**
     * Bootstrap the service provider.
     *
     * @return void
     */
    public function boot()
    {
        Socialite::extend('line', function ($app) {
            $config = $app['config']['services.line'];

            return Socialite::buildProvider(LineProvider::class, $config);
        });
    }
}
```

Facade を使いたくない場合はこうやればいいはず。

```
$socialite = $this->app->make(Factory::class);
$socialite->extend
```


Manager ごと入れ替えなくても Socialite::extend で元の Manager のまま拡張できる。
公式見解として Instagram 対応へのプルリクにも `extend` 使えよ😁と答えてる。
https://github.com/laravel/socialite/pull/27


`boot()` に書くのが大事。`register()` ではだめ。

パッケージにしない場合は ServiceProvider も不要で boot() の中身を AppServiceProvider にでも書けば十分。

## $defer = false
後から気付いたけどServiceProviderで `$defer = false` にしておかないと複数の独自ドライバーを使ってる場合に最後に読み込まれたドライバーしか有効にならない。

## 使う時
Laravel 5.5 以降なら ServiceProvider の登録は自動なので `composer require` でインストールして `config/services.php` と `.env` で設定すればすぐ使える。

```
return Socialite::driver('line')->redirect();
```

## おまけ
Manager クラスは Laravel 内でも色々使われてて使い方覚えると自分のアプリ内でも便利に使える。記事書きたいけど説明が難しい…。
