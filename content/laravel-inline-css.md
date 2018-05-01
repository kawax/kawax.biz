---
title: "LaravelでインラインCSS"
date: 2018-04-30T23:41:53+09:00
categories: ["Laravel"]
draft: false
---

PageSpeed Insightsの点数を上げるだけならインラインCSS化が一番効く。
HTTP/2 Server Pushの時代に意味があるかはともかく。
どちらがいいかを試すためにインラインCSSもできるようにしておく。

`@inline_css('css/app.css')`みたいに使えるようにBladeで拡張すればいいと思って作りかけたけど
https://readouble.com/laravel/5.6/ja/blade.html#extending-blade

```php
        Blade::directive('inline_css', function ($path) {
            $css = File::get(public_path(trim($path, "'")));

            return '<style type="text/css">' . $css . '</style>';
        });
```

作ってる途中で$pathに`'`が含まれてることに気付く。`'css/app.css'`で渡される。
そういう仕様？

Blade拡張を集めたパッケージを調べたらそれでも`'`を削除してた。仕様だ。
https://github.com/appstract/laravel-blade-directives/blob/47424072001279063e69fd117e4fe1c1e4e5ce2d/src/DirectivesRepository.php#L35

```php
    public static function stripQuotes($expression)
    {
        return str_replace("'", '', $expression);
    }
```

そしてここで`@inline`があるのに気付く。自分で作らなくてもこれ使えばいい。
https://github.com/appstract/laravel-blade-directives#inline


インラインにすべきは小さいCSSファイルとクリティカルCSSなのでapp.cssをまるごとインラインにするのは良くない。
かと言ってクリティカルCSSを手動で分離なんてやるわけないのでインラインCSSは使える方法じゃなさそう。
やはりServer Pushか。
