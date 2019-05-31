---
title: "Laravelのコーディング規約"
date: 2019-05-31T13:38:16+09:00
categories: ["Laravel"]
draft: false
---

基本的にはPSR-2 + PHPDoc部分の独自規約程度だけど

- https://readouble.com/laravel/5.8/ja/contributions.html#coding-style
- https://laravel.com/docs/5.8/contributions#coding-style

実際は他にも色々。クラスのインポートは長さ順とか。

```php
use Illuminate\Foundation\Mix;
use Illuminate\Support\HtmlString;
use Illuminate\Container\Container;
use Illuminate\Support\Facades\Date;
...
use Illuminate\Contracts\Validation\Factory as ValidationFactory;
use Illuminate\Contracts\Broadcasting\Factory as BroadcastFactory;
```

- https://github.com/laravel/framework/blob/fe9c6169dfe2c3ebf1b3769b10585d9f9f452f0f/src/Illuminate/Foundation/helpers.php


PhpStormならOptimize Importsで同じようにできるのでどこかでは使われてるルールなんだろう。

- https://pleiades.io/help/phpstorm/creating-and-optimizing-imports.html

これが見やすいか？と考えると別に見やすくないので自分ではあまり使わない。機械的にフォーマットするならこういうルールが必要なんだろうけど。

自動化するならStyleCIでできるけどLaravelだと厳しいのでPSR-2に合わせてる。

- https://styleci.io/

## 重要なこと
この規約はLaravel公式内で適用される規約であってLaravel使って何か作るユーザーが従う必要はない。
PSR-2さえ無視してもいい。さすがに今時PSR-2無視してたら他人が読めないけど。

個人的な落とし所。

- PhpStormのLaravelプリセットで自動フォーマット。
- StyleCIはPSR-2

他人のプロジェクトなら合わせるけど一目見ただけで分かるほどひどいコードで規約も何もないような状態なら全部フォーマットし直す。
Laravel使ってるのにarray()使ってるような人は本当に実在する。array()なんてもはやフォーマットするだけで問答無用で[]に変換される。
こんなレベルに合わせるのは不可能なので容赦なくフォーマット。