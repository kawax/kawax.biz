---
title: "Laravel5.6の準備"
date: 2018-01-27T13:34:06+09:00
categories: ["Laravel"]
draft: true
---

目立った新機能はないけど本当に来月出るのかな…？と思うけど今のLaravelの開発は半年ごとのリリースは決まってて
BREAKING CHANGEなら次のバージョン、違うなら現在のバージョン、で振り分けて取り込んでるだけなので気にしても仕方ない。

## まず5.6にするか決める
5.5は長期サポート版なのでもうしばらく使い続けてもいい。PHP7.1が用意できないとか半年ごとのリリースペースが早すぎて着いて行けねぇよって場合は留まるのもあり。

5.1の頃はまだどんどん追加されてたけど5.5なら大分落ち着いてる。

自分はここの所ほいほい新プロジェクト作りすぎたので5.5のまま残すのもあるはず。

## Bootstrap 4
5.6はLaravelよりもBootstrap4への移行のほうが大変。3から移行するつもりの人は覚悟したほうがいい…。3のままか4にするか決めておく。

## ページネーション
Bootstrap4への移行でページネーションにも影響が出る。
ここ直して欲しかったけど無理だった。
やはり英語での説明が難しい…。
Bootstrap3/4/他のCSSを5.5 / 5.6でどう使うかで色々なのでややこしい。

今のまま行くと

### bootstrap4
影響なくそのまま使える。

5.5のAppServiceProviderでこんな設定してる場合は5.6で消せる。

```php
Paginator::defaultView('pagination::bootstrap-4');
Paginator::defaultSimpleView('pagination::simple-bootstrap-4');
```

### bootstrap3
少し対応作業が必要。でも実際は何もしなくても問題ないのでほとんどの人は気付かず使いそう。

### 他のCSS
`views/vendor/pagination/default.blade.php` にカスタムページネーション置いて使ってる場合はぶっ壊れる。

AppServiceProviderで設定が必須。元から`default.blade.php`の名前でなくここの設定してるなら問題ないけどあまり見ない。

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

自分の案ではどの場合でも対応作業は不要でどうしてもbootstrap3を厳密に使いたい人のみこうすればいいよ、とやりかったけど無理だった。

これでカスタムページネーション使ってる全プロジェクトでBREAKING CHANGEが発生…。

## その他
LogがManagerベースになってたりするけど普通に使う分には関係ないだろう。
