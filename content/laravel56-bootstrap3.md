---
title: "Laravel 5.6 のページネーションで Bootstrap3 を使う"
date: 2018-01-22T11:15:33+09:00
categories: ["Laravel"]
draft: false
---

https://github.com/laravel/framework/commit/12d789de8472dbbd763cb680e896b3d419f954c0

Laravel5.6ではbootstrap4がデフォルトになるけど3から4への移行は結構面倒なので3のまま使う人も多いと思う。
そのままではページネーションだけbootstrap4が使われるので3にするにはAppServiceProvider辺りで


```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;

use Illuminate\Pagination\Paginator;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Paginator::defaultView('pagination::default');
        Paginator::defaultSimpleView('pagination::simple-default');
    }
```

5.5で設定してても大丈夫なので今のうちにしておけば後で気にしなくていい。

（追記）でもこれ `views/vendor/pagination` でカスタムページネーションを使ってる場合も必要になるような…。影響が大きいので5.6リリースまでに変更されるかもしれない。
