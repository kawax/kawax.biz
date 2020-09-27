---
title: "Laravel8 新しいルーティングの書き方"
date: 2020-09-14T20:24:50+09:00
categories: ["Laravel"]
draft: false
---

## はじめに
Laravel8ではルーティングの書き方が若干変更されている。
慣れてる人や旧バージョンからアップグレードした環境では影響は少ないけど
Laravel8リリース後に使い始めた初心者は大混乱している。

旧バージョン時の情報は使えなくなってるので新しい書き方を覚える必要がある。

## 読むところ
- https://laravel.com/docs/8.x/releases
- https://laravel.com/docs/8.x/upgrade
- https://laravel.com/docs/8.x/routing
- https://readouble.com/laravel/8.x/ja/releases.html
- https://readouble.com/laravel/8.x/ja/upgrade.html
- https://readouble.com/laravel/8.x/ja/routing.html

## Laravel7までは
こう書けた。

```php
Route::get('/user', 'UserController@index');
```

これで良かった理由は`app/Providers/RouteServiceProvider.php`で$namespaceが定義されていたから。

```php
    /**
     * This namespace is applied to your controller routes.
     *
     * In addition, it is set as the URL generator's root namespace.
     *
     * @var string
     */
    protected $namespace = 'App\Http\Controllers';
```

`UserController`だけ書いても`App\Http\Controllers`が追加されて`App\Http\Controllers\UserController`となる。

（そもそも名前空間を省略できるようにしてるのは名前空間使ってなかったLaravel4頃の名残り？）

例外はarrayでの指定時。callable syntax。

```php
use App\Http\Controllers\UserController;
Route::get('/user', [UserController::class, 'index']);
```

## Laravel8では
例外だった書き方が標準。

```php
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

RouteServiceProviderの$namespaceがコメントアウトされたので常に名前空間まで含むクラス名で指定。
「コントローラーをuseして`::class`で指定」と覚えておけばいいのでは。

この変更によりシングルアクションコントローラーはこう書けるようになった。

```php
Route::get('/user', UserController::class);
```

Laravel7までは↑はできそうでできない。以下のどちらか。

```php
Route::get('/user', [UserController::class, '__invoke']);
Route::get('/user', 'UserController');
```

Laravel8で`UserController`だけ書いても`App\Http\Controllers`が追加されないのでクラスが存在しないエラーとなる。

## Laravel8でどうしても旧バージョンの書き方したい場合
アップグレードガイドに書かれてるように`RouteServiceProvider`で$namespace使うように変更すればいい。

```php
class RouteServiceProvider extends ServiceProvider
{
    /**
     * This namespace is applied to your controller routes.
     *
     * In addition, it is set as the URL generator's root namespace.
     *
     * @var string
     */
    protected $namespace = 'App\Http\Controllers';

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        $this->configureRateLimiting();

        $this->routes(function () {
            Route::middleware('web')
                ->namespace($this->namespace)
                ->group(base_path('routes/web.php'));

            Route::prefix('api')
                ->middleware('api')
                ->namespace($this->namespace)
                ->group(base_path('routes/api.php'));
    });
}
```

旧バージョンからのアップグレード時はRouteServiceProviderを変更しなければそのまま使える。
新しい書き方に変えるかどうかは自由。

逆に、Laravel7以前でもRouteServiceProviderからnamespaceを削除すればLaravel8の書き方ができる。Laravel6(LTS)使ってるけど今後のためにLaravel8に合わせるのはあり。

## 変更した理由？
不明なもののJetstreamで使うLivewireのため？
https://laravel-livewire.com/docs/2.x/upgrading

シングルアクションコントローラーが便利になったのは偶然な気がする。
毎度同じだけど個人的には「リソースコントローラー以外は全部シングルアクションコントローラーで作る」が推奨。
これならarrayもなく全部`::class`の指定のみになる。

```php
Route::get('user/{id}', ShowProfile::class);
Route::resource('photos', PhotoController::class);
```

`'UserController'`だとうっかりtypoすることもあるだろうし`UserController::class`のほうがミスは少ない。
IDEフレンドリー。

## ついで
Laravel8からクロージャを使ったルーティングも`php artisan route:cache`できるようになっている。
デフォルトのこれのままでもキャッシュできる。

```php
Route::get('/', function () {
    return view('welcome');
});
```

`routes/web.php`は修正してもapi.phpを忘れてて最初のデプロイ後に修正、が定番だったけどそれもなくなる。