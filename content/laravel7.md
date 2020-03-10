---
title: "Laravel7へアップグレード"
date: 2020-03-04T11:28:58+09:00
categories: ["Laravel"]
draft: false
---

## アップグレードの前に
Laravel 6.xはLTSなのでまだ使い続けてもいい。
7に上げるなら今後も半年毎のバージョンアップについていく必要がある。
重要な分岐点なのでよく考えて決める。

個人的にはプロジェクト増えすぎて完全に限界なのでなるべく6のまま残してCORSが必要なプロジェクトだけ7にする予定。
CORS使ってるってことはそれなりの規模なので手間としてはそんなに変わらないけど。

## いつもの見る所
https://laravel.com/docs/7.x/releases
https://laravel.com/docs/7.x/upgrade
https://github.com/laravel/laravel/compare/6.x...master

非公式日本語版は書いてる時点ではまだない。
https://readouble.com/laravel/7.x/ja/releases.html
https://readouble.com/laravel/7.x/ja/upgrade.html

ファイル追加やコピペのためにzipでダウンロードしておく。
https://github.com/laravel/laravel

## いつものようにサードパーティパッケージの対応待ち
ファーストパーティ=Laravel公式は対応してるけど非公式は対応してないことが多いだろうから待つか自分でプルリクかdev用で必須ではないなら一時的に不使用にするとか対応を決める。

## 今回は
Laravel4.0からのプロジェクトを6→7に上げた段階で書いた記事。
こういう古いプロジェクトはアップグレードを試すのにちょうどいい。

これ元はLaravelの前に別のフレームワークで作ってたのをLaravel4.0で作り直し、5.0移行時にまた作り直してる。
古い箇所は残ってるけどしっかりバージョンアップさえしていれば問題なく動く。

## ^7.0
とりあえずcomposer.jsonを書き換えて`composer update`しても早速サードパーティパッケージで引っかかる。
一時的に消しても影響ないので消して進める。

## Symfony 5
`App\Exceptions\Handler`を修正。
configは後でまとめて。

別プロジェクトでの余談。Symfonyは3,4,5のコンポーネントが混在してても平気で動く。

## Blade
Blade::component()から`Blade::aliasComponent()`に変更。
ここでは使ってないけど他で使ってるので後で。

## プロジェクト側
`composer update`さえ通ればフレームワーク側はあまり変更ないかも。
これも毎回同じだけど「使ってる機能次第」なのでプロジェクトによる。

差分見ながらプロジェクト側を修正していく。
https://github.com/laravel/laravel/compare/6.x...master
コピペしていけばいいだけなんだけど厄介なことに6.x期間中も細々と変更されててこれだけでは気付かないかもしれない。CHANGELOGも見たほうがいい。
https://github.com/laravel/laravel/blob/master/CHANGELOG.md

## app/Http/Kernel.php
`\Fruitcake\Cors\HandleCors::class,`追加と`$middlewarePriority`削除。
$middlewarePriorityはいつの間にか増えてたけどよく分からないまま消えた。

## 終わり
なにか気付いたら後で追記。

## Date Serialization

7に上げた後しばらくしてから気付いたけどこれは確かに「影響の可能性：高」
6までは`2019-12-02 20:01:00`だけど7からは`2019-12-02T20:01:00.283041Z`の形式。
影響があるのはAPI Resources使わずに直接Eloquentを返してjson化してるような所。
Laravelの便利な機能ではあるけどこれを使ってると日時の形式が変わる。

```php
return $posts;
```

6までと同じ形式に戻すならEloquentモデルにserializeDate()追加。

```php
/**
 * Prepare a date for array / JSON serialization.
 *
 * @param  \DateTimeInterface  $date
 * @return string
 */
protected function serializeDate(\DateTimeInterface $date)
{
    return $date->format('Y-m-d H:i:s');
}
```

jsonを受け取った側でそのまま表示してるだけなら見た目が変わるだけで影響は少ない。
serializeDate()を追加せず新形式のままJavaScript側で解析して好きな形式で表示する手もある。
Laravel公式としてはこの使い方を想定。