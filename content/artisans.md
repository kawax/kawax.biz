---
title: "Laravel職人を探すサービスを作った"
date: 2019-03-16T19:06:31+09:00
categories: ["Laravel"]
draft: false
---

現状は職人を募集してる段階。
https://artisans.kawax.biz/


ユーザーごとのページはこんな。
https://artisans.kawax.biz/@kawax


技術的にはLaravel+Vue.jsを**普通**に使ってるだけなので何もない。
このくらいの小規模なものは無駄に難しいことせずささっと作ればいい。
開発環境もSQLiteと`php artisan serve`だけで最小限。
今後キューが必要になったら変わるかもしれないけど。


それでも今回初めて気付いたこと。

## Markdown
Laravelのメール機能でMarkdown使うためにすでにこれがインストールされている。
https://github.com/erusev/parsedown
LaravelでMarkdown表示するならこれをそのまま使うのが早い。
https://github.com/laravel/framework/blob/94cc6a46dc7b000fb1774da9ca4c8610f31392b0/src/Illuminate/Mail/Markdown.php#L88

Bladeで。

```php
{{ Illuminate\Mail\Markdown::parse(e($text)) }}
```

さらにカスタムディレクティブ`@markdown()`で使えるようにするならServiceProviderで登録。

```php
use Illuminate\Support\Facades\Blade;

Blade::directive('markdown', function ($text) {
    return "<?php echo Illuminate\Mail\Markdown::parse(e($text)); ?>";
});
```

```php
@markdown($text)
```

## Buefy
Vue使ってるのはフォームのみ。
https://buefy.org/
タグの入力フォームとか自分で作るものじゃない。
https://buefy.org/documentation/taginput/

CSSは最近Bulmaが増えてるけどJSは含まれてないのでNavbarでさえ自分で書かないと動かない。
https://bulma.io/documentation/components/navbar/

ページネーション用のファイル作ったので置いておく。
https://github.com/kawax/laravel-pagination-bulma
