---
title: "Salvager: Laravel Duskベースのスクレイピングライブラリ"
date: 2019-01-02T22:35:33+09:00
categories: ["Laravel"]
draft: false
---

発想としてはGoutte(Guzzle+DomCrawler)と同じ。Laravel Dusk(Headless Chrome)+Symfony DomCrawler。
https://github.com/FriendsOfPHP/Goutte

最初はLaravel Console Dusk使ってたけどCommandからしか使いにくいしChrome Driverのオプション変更がちょっと手間だったので自分で作った。
https://github.com/nunomaduro/laravel-console-dusk

Headless ChromeなのでJavaScriptが必要なGoutteで読めない所でも対応できる。サーバー上で動かすにはChromeのインストールが必要なので当然ながらその辺のレンタルサーバーでは動かない。Laravel使ってる人がレンタルサーバー使ってるわけないのでそんなレベルのことは想定してない。

https://github.com/kawax/salvager

## 必要な環境
- PHP 7.1以上
- 最新のChrome。Linux版のインストール方法は各自で検索。

## Laravelでの使い方
composerでインストールしてFacadeでどこからでも使えばいいだけ。自分でこうできるようにするために作ったので余計な作業は一切ない。

```
composer require revolution/salvager
```

```php
use Laravel\Dusk\Browser;
use Symfony\Component\DomCrawler\Crawler;

use Revolution\Salvager\Facades\Salvager;

class SalvagerController
{
    public function __invoke()
    {
        Salvager::browse(function (Browser $browser) use (&$crawler) {
            $crawler = $browser->visit('https://www.google.com/')
                               ->keys('input[name=q]', 'Laravel', '{enter}')
                               ->screenshot('google-laravel')
                               ->crawler();
        });

        /**
         * @var Crawler $crawler
         */
        $crawler->filter('.r')->each(function (Crawler $node) {
            dump($node->filter('h3')->text());
            dump($node->filter('a')->attr('href'));
        });
    }
}
```

`Salvager::browse()`内はDuskと同じ。
https://readouble.com/laravel/5.7/ja/dusk.html
最後に`crawler()`でDomCrawlerのインスタンスを返す。
https://symfony.com/doc/current/components/dom_crawler.html
やってることはGoutteと同じでシンプル。中身見ずにGoutteがスクレイピングのための特殊なことしてると思ってる人がいるけど実体はGuzzleとDomCrawlerの組み合わせ。使い慣れてるだろうDomCrawlerから変える理由はないのでそのまま使用。変更したいなら`Laravel\Dusk\Browser::macro()`で好きに増やせばいい。


crawler部分の処理は無名関数の外で続けるために`use (&$crawler)`としている。現代PHPで唯一と言ってもいい参照渡し使う場面。Laravelならこう使うことが多くなるはず。

Duskと同じで複数ブラウザの起動もできるのでGoutteのように`browse()`からcrawlerを返す仕様にはしてない。

```php
Salvager::browse(function (Browser $browser, Browser $browser2) use (&$crawler, &$crawler2) {
    $crawler = $browser->visit('https://example.com/')
                       ->crawler();

    $crawler2 = $browser2->visit('https://example.org/')
                         ->crawler();
});
```

### config
```
php artisan vendor:publish --provider="Revolution\Salvager\Providers\SalvagerServiceProvider"
```

で`config/salvager.php`が作られるけどDocker内で使わないなら不要なはず。

### Facadeを使わない場合
サービスコンテナ経由で。Facade使わない人に細かい説明はいらない。

```php
use Revolution\Salvager\Client;
use Revolution\Salvager\Contracts\Factory;

class SalvagerController
{
    public function __invoke(Client $client)
    {
        $client = app(Factory::class);

        $client->browse(function (Browser $browser) use (&$crawler) {
          //
        });
    }
}
```

## 素のPHPでの使い方
一応Laravel以外でも使える。

### Dockerを使った例
Chromeインストールしてない環境でも動くように。
3行コピペ。

```
git clone https://github.com/kawax/salvager.git salvager && cd $_

docker-compose run --rm composer install

docker-compose run --rm example google.php
```

### google.phpの中身
LaravelならServiceProviderに押し込める初期化作業を自分で行う。

```php
<?php
require_once __DIR__ . '/../vendor/autoload.php';

use Laravel\Dusk\Browser;
use Symfony\Component\DomCrawler\Crawler;

use Revolution\Salvager\Client;
use Revolution\Salvager\Drivers\Chrome;


Browser::$storeScreenshotsAt = __DIR__ . '/screenshots/';
Browser::$storeConsoleLogAt = __DIR__ . '/console/';

Browser::macro('crawler', function () {
    return new Crawler($this->driver->getPageSource() ?? '', $this->driver->getCurrentURL() ?? '');
});

$options = [
    '--disable-gpu',
    '--headless',
    '--window-size=1280,720',

    // Docker
    '--no-sandbox',
];

$client = new Client(new Chrome($options));

$client->browse(function (Browser $browser) {
    /**
     * @var Crawler $crawler
     */
    $crawler = $browser->visit('https://www.google.com/')
                       ->keys('input[name=q]', 'PHP', '{enter}')
                       ->screenshot('google-php')
                       ->crawler();

    $crawler->filter('.r')->each(function (Crawler $node) {
        dump($node->filter('h3')->text());
        dump($node->filter('a')->attr('href'));
    });
});
```

ここでは`browse()`内でそのまま$crawlerを使ってる。

## 自分の使い方
artisanコマンドとして作って定期実行させてる。スクレイピングよりはスクリーンショット目的。
GitLab CI+Dockerで運用するとサーバーレス。

AWSやGCPのUbuntuサーバーでも動かしてるけど割と失敗するのでDockerのほうがいいかも。

## 終わりと注意
相手サーバーの負荷を考えない過度なスクレイピングはしないように。
最近特にPython初心者による非常識なスクレイピングが目立つ。
