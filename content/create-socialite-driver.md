---
title: "Laravel Socialite 独自ドライバーの作り方・改"
date: 2019-12-18T11:21:44+09:00
categories: ["Laravel"]
draft: false
---

前に書いたけどQiita版は後から修正しないのでまた書く。

## 環境
- Socialite 4.3.1
- Laravel 6.x
- PHP 7.4

## SocialiteProvidersは非公式
Laravelのドキュメントに書かれているけどSocialiteProviders自体はコミュニティによる非公式なもの。サードパーティパッケージ。
https://socialiteproviders.netlify.com/

最近見かけたfreeeのPHP SDK
https://github.com/freee/freee-accounting-sdk-php
これもSocialiteProviders使ってる。Socialite自体は問題なく動くとは思うけどその後Auth周りの色々やっている部分がなんでこんなことしてるのかさっぱり分からない。（たぶんDB使わないようにしてるだけ）

こんなことしなくても公式のSocialiteを拡張する形で作れば簡単。

## Laravelプロジェクト内に作る
一番シンプルに既存のプロジェクト内に作る方法。

`app/Socialite/FooProvider.php`を作る。

```php
namespace App\Socialite;

use Laravel\Socialite\Two\AbstractProvider;
use Laravel\Socialite\Two\ProviderInterface;
use Laravel\Socialite\Two\User;

class FooProvider extends AbstractProvider implements ProviderInterface
{
    //
}
```

中身は公式からコピペして書き換えればいい。URLとmapUserToObject()が主。
https://github.com/laravel/socialite/tree/4.0/src/Two

次に`AppServiceProvider@boot()`

```php
use Illuminate\Support\ServiceProvider;
use Laravel\Socialite\Facades\Socialite;
use App\Socialite\FooProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Socialite::extend(
            'foo',
            function ($app) {
                return Socialite::buildProvider(FooProvider::class, config('services.foo'));
            }
        );
    }
}
```

PHP7.4なら1行で書けるかも。

```php
Socialite::extend('foo', fn($app) => Socialite::buildProvider(FooProvider::class, config('services.foo')));
```

後は`config/services.php`と`.env`で設定。

```php
    'foo' => [
        'client_id'     => env('FOO_ID'),
        'client_secret' => env('FOO_SECRET'),
        'redirect'      => env('FOO_REDIRECT'),
    ],
```

これで`Socialite::driver('foo')`で使えるようになる。

作ったのはFooProviderの1ファイルだけ。
SocialiteProvidersのなぜかイベント使ってるような箇所は不要。

## composerパッケージに分離
最低限必要なファイルは3つ。

- FooProvider.php
- FooServiceProvider.php
- composer.json

FooServiceProviderが前は少し間違ってた。複数の独自ドライバーを同時に使った時のみ発生するので気付きにくかった。
LaravelやSocialiteのバージョンの影響も受けるけど現時点のLaravel6とSocialite4.3.1ならこれでいい。

```php
namespace My\Socialite\Foo;

use Illuminate\Support\ServiceProvider;
use Laravel\Socialite\Facades\Socialite;

class FooServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap the service provider.
     *
     * @return void
     */
    public function boot()
    {
        Socialite::extend(
            'foo',
            function ($app) {
                $config = $app['config']['services.foo'];

                return Socialite::buildProvider(FooProvider::class, $config);
            }
        );
    }
}
```

テストは公式を参考に真似すればいい。
https://github.com/laravel/socialite/blob/4.0/tests/OAuthTwoTest.php