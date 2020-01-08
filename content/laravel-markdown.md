---
title: "LaravelでMarkdownをhtmlに変換・改"
date: 2020-01-08T11:40:48+09:00
categories: ["Laravel"]
draft: false
---

Laravel v6.10.0でparsedownから`league/commonmark`に変わったので再度。
https://github.com/thephpleague/commonmark

`Illuminate\Mail\Markdown::parse()`はLaravel内でのMail用なのでそのまま使うとhtmlがエスケープされずに危険。これを元に自分のプロジェクト用に作ればいい。

まずオートリンク用に`league/commonmark-ext-autolink`をインストール。その他の必要なパッケージは6.10以降ならLaravel側でインストールされている。

```
composer require league/commonmark-ext-autolink
```

`app/Support/Markdown.php`を作る。

```php
<?php

namespace App\Support;

use Illuminate\Support\HtmlString;
use League\CommonMark\CommonMarkConverter;
use League\CommonMark\Environment;
use League\CommonMark\Ext\Table\TableExtension;
use League\CommonMark\Ext\Autolink\AutolinkExtension;

class Markdown
{
    /**
     * Parse the given Markdown text into HTML.
     *
     * @param  string  $text
     *
     * @return HtmlString
     */
    public static function parse($text)
    {
        $environment = Environment::createCommonMarkEnvironment();

        $environment->addExtension(new TableExtension());
        $environment->addExtension(new AutolinkExtension());

        $converter = new CommonMarkConverter([
            'html_input'         => 'escape',
            'allow_unsafe_links' => false,
        ], $environment);

        return new HtmlString($converter->convertToHtml($text ?? ''));
    }
}
```

htmlを完全に消すなら`'html_input' => 'strip',`でもいい。


使う時は

```php
use App\Support\Markdown;

$html = Markdown::parse($markdown);
```
