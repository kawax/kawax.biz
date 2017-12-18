---
title: "BotMan for ChatWork"
date: 2017-12-18T21:05:19+09:00
categories: ["Laravel"]
draft: false
---

作り始めた所。

## BotMan
`The only PHP chatbot framework you will ever need.`  
https://botman.io/

BotMan を Laravel に導入済にして用意してあるのが BotMan Studio  
https://github.com/botman/studio

今回は BotMan Studio を元に作っていった。

## ChatWork Webhook
BotMan のデフォルトでは当然 ChatWork はない。ChatWork はこういう所が弱すぎる。少し前に Webhook 対応してやっと Chat bot 的なものを作れるようになった。

http://developer.chatwork.com/ja/webhook.html

調べた結論としては Webhook は `アカウントイベント` と `ルームイベント` があり

- `アカウントイベント` に `ご自身へのメンション(mention_to_me)`
- `ルームイベント` に `メッセージ作成(message_created)` と `メッセージ更新(message_updated)`

一つの Webhook 設定で両方のイベントが飛んでくることはないので両方使うなら Webhook を2つ作る。
bot へのメンション時にのみ発動させるなら `アカウントイベント` だけでもいい。
両方使うと bot へのメンションで mention_to_me と message_created 両方発動するので一つのほうがいいかもしれない。

## 作ったものとデモ
- https://www.chatwork.com/g/botman
- https://github.com/kawax/botman-chatwork
- https://github.com/kawax/botman-chatwork-project
- https://botman.kawax.biz/

## 基本的な使い方
導入は GitHub 見てなんとかしてもらうとして…
Chat bot としての動作をどこに書くか。

書く場所としては2つ。

### routes/botman.php

```php
<?php

use App\Http\Controllers\BotManController;

$botman = resolve('botman');

$botman->hears('Hi', function ($bot) {
    $bot->reply('Hello!');
});
```

Laravel のルーティングと大体同じなので分かりやすい。

クロージャではなくクラスで指定もできる。クラスの置き場所はどこでもいい。

```php
$botman->hears('laravel version', 'App\Botman\LaravelCommand@version');
```

bot へのメンションで `laravel version` と書けば GitHub API を使って取得した Laravel の最新バージョンを返すコマンド。
Laravel ベースだとなんでもできるのがメリット。Lambda とかでも作れるけど不慣れだと後からメンテできなくなる可能性が高い。

### webルーティングからコントローラー
`routes/web.php` の

```php
Route::match(['get', 'post'], '/botman', 'BotManController@handle');
```

コントローラー内に書いている。

```php
    /**
     * Place your BotMan logic here.
     */
    public function handle()
    {
        $botman = app('botman');

        //$botman->hears();

        $botman->listen();
    }
```

webhook で指定するのは `/botman` のURL。

どっちかを使うかは別にどっちでもいいはずだけどたぶん botman.php。`/botman` へのリクエストで BotMan が発動(listen)、実際の動作の細かいことは botman.php。
