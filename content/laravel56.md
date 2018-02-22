---
title: "Laravel5.6へアップグレード"
date: 2018-02-08T08:19:21+09:00
categories: ["Laravel"]
draft: false
---

結局大きな新機能はないまま5.6リリース。その分更新作業もすぐ終わるけど慌てなくていい。

## まず5.6にするか決める
5.5は長期サポート版なのでもうしばらく使い続けてもいい。PHP7.1が用意できないとか半年ごとのリリースペースが早すぎて着いて行けねぇよって場合は留まるのもあり。

5.1の頃と比べると5.5で困ることもないだろうし。

5.6にしたらその後もバージョンアップを追い続けることになる。

## 準備
- https://readouble.com/laravel/5.6/ja/releases.html
- https://readouble.com/laravel/5.6/ja/upgrade.html
- https://laravel.com/docs/5.6/releases
- https://laravel.com/docs/5.6/upgrade
- https://github.com/laravel/laravel/compare/5.5...master
- https://github.com/laravel/framework/blob/5.6/CHANGELOG-5.6.md

これ見ればいいだけなので書くことはあまりない。

## composer.json
色々メジャーバージョンが上がってるので書き換える。

書かれてないけど昔からのプロジェクトではまだ使ってるかもしれないBrowserKit Testingは4.0が出てない。
https://github.com/laravel/browser-kit-testing

## config
`hashing.php` と `logging.php` をコピーしてくる。
https://github.com/laravel/laravel/tree/master/config

## Trusted Proxies
前はarrayだったけど指定方法が変わった。

```php
protected $headers = Request::HEADER_X_FORWARDED_ALL;
```

## ページネーション
これなー。なるべく少ない手間で更新できる案があったけど上手く説明できずに失敗した…。

### Bootstrap 4
今回はLaravelよりもBootstrap3から4への移行が一番大変だから移行するつもりの人は覚悟したほうがいい…。

5.5でこう設定して使ってた場合は消せる。

```php
Paginator::defaultView('pagination::bootstrap-4');
Paginator::defaultSimpleView('pagination::simple-bootstrap-4');
```

4初期から使っててカスタムページネーションで対応してる場合も消せる。

### Bootstrap 3
こう設定するように書いてるけど実はなくても影響は少ない。Bootstrap3から4はclassが少し増えただけなので3で自分でclass作ってなければ4用のページネーションで問題なく表示できる。

```php
Paginator::useBootstrapThree();
```

### 他のCSS + カスタムページネーション
問題はここ。そのままだとぶっ壊れるはず。
カスタムページネーションは `views/vendor/pagination/default.blade.php` に置いて使う。`default.blade.php` の名前に依存している。これを5.6でbootstrap4をデフォルトにするために安易に `bootstrap-4.blade.php` に変更しただけだからカスタムページネーションに影響が出る。

対応としてはdefaultViewで設定。

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
        Paginator::defaultView('vendor.pagination.default');
    }
```

最近はBootstrap以外を使うことが多いから全部でこの設定することになって面倒…。
解決案としては `default.blade.php` をBootstrap4用にして3用には `bootstrap-3.blade.php` を作る。
これが一番影響少ないはず。Bootstrap 3を厳密に使いたい人以外は何もしなくていい。
5.6はBootstrap 4がデフォルトなので優先度は4>=カスタムページネーション>3。

アップグレードガイドにも書いてないくらいなので全く考慮してない…。

## 終わり
無理に更新する必要はなさそう。
