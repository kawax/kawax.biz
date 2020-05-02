---
title: "LaravelでMarkdownをhtmlに変換・改"
date: 2020-01-08T11:40:48+09:00
categories: ["Laravel"]
draft: false
---

Laravel v6.10.0でparsedownから`league/commonmark`に変わったので再度。
https://github.com/thephpleague/commonmark

`Illuminate\Mail\Markdown::parse()`はLaravel内でのMail用なのでそのまま使うとhtmlがエスケープされずに危険。これを元に自分のプロジェクト用に作ればいい。

Laravel側で急に`league/commonmark`が消えてもいいように一応自分のプロジェクトでもインストール。
```json
    "league/commonmark": "^1.4",
```

`app/Support/Markdown.php`を作る。

```php
<?php

namespace App\Support;

use Illuminate\Support\HtmlString;
use League\CommonMark\GithubFlavoredMarkdownConverter;

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
        $converter = new GithubFlavoredMarkdownConverter([
            'html_input'         => 'escape',
            'allow_unsafe_links' => false,
        ]);

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

HtmlStringなのでviewでは`{{ $html }}`でもそのまま表示される。入力されたhtmlはエスケープ済なので変換後のhtmlは安全な前提。

## 追記
league/commonmark1.3以降用に`GithubFlavoredMarkdownConverter`使うように変更。