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

## （さらに追記）実際にやってみた
こういうのは試してみないと分からないので5.6.x-devとBootstrap3で試した。
結果としては **5.6のBootstrap4用のページネーションで問題なく表示できる**
Bootstrap3->4での変化がclassを追加するだけなので3では意味のないclassが付いてるだけなので影響はない。
5.6でBootstrap3を使い続ける場合でもページネーションのことは気にしなくていい。


カスタムページネーションを使ってる場合には影響がある。
5.5では `views/vendor/pagination/default.blade.php` に置けば使われたけど5.6ではAppServiceProviderでの設定も必要。

```php
Paginator::defaultView('vendor.pagination.default');
```

ファイル名を `bootstrap-4.blade.php` にしてもいいけどBootstrap4使ってないのにこの名前は違和感。

ということは…Laravel側の `bootstrap-4.blade.php` を `default.blade.php` にしてもらえれば全部綺麗に収まる。
