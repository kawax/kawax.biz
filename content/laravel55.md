---
title: "Laravel5.5へアップグレード"
date: 2017-08-31T16:48:42+09:00
categories: ["Laravel"]
draft: false
---

いつものアップグレード作業の時期。
ざっと見たところ大きな変更はないので5.4からなら簡単そう。

<!--more-->

関わってるLaravelプロジェクトが増えてきてさすがに大変になってきた…。  

すでに日本語ドキュメントも翻訳中なので参考に。

- https://readouble.com/laravel/5.5/ja/releases.html
- https://readouble.com/laravel/5.5/ja/upgrade.html
- https://laravel.com/docs/5.5/releases
- https://laravel.com/docs/5.5/upgrade

## LTS
5.5は5.1以来の長期サポートということになっている。
長期サポートと言ってもまさか5.1を2,3年も使うわけないのでLaravel使うなら毎回バージョンアップするのが基本。
5.1から5.5へとか…やりたくない。

## PHP7.0以上が必須
5.5はまずここが一番大きな変化。PHP5.6のサポートが来年まで残ってる段階でのPHP7移行は正直早すぎるけどPHP7使いたい気持ちはよく分かる。
半年前に告知出てたのでLaravel5.5使うような人なら問題なく対応できるだろう。

## プロジェクトファイルの差分
5.4の時にあった比較ツールへのリンクがないけどたぶんこれでいいはず
https://github.com/laravel/laravel/compare/5.4...master
  
ファイル追加したりコピペしたりするのでプロジェクトをzipでダウンロードしておくと楽。

## composer.json

バージョン上がったり追加されたり変更されたり色々なのでよく見て書き換える。
https://github.com/laravel/laravel/blob/master/composer.json

```json
    "require": {
        "php": ">=7.0.0",
        "fideloper/proxy": "~3.3",
        "laravel/framework": "5.5.*",
        "laravel/tinker": "~1.0"
    },
```

```json
    "require-dev": {
        "filp/whoops": "~2.0",
        "fzaninotto/faker": "~1.4",
        "mockery/mockery": "0.9.*",
        "phpunit/phpunit": "~6.0"
    },
```

```json
    "extra": {
        "laravel": {
            "dont-discover": [
            ]
        }
    },
```

```json
    "scripts": {
        "post-root-package-install": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "@php artisan key:generate"
        ],
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover"
        ]
    },
```

他のパッケージが5.5に対応してなかったらここで中断することになる。
バージョン指定で `"illuminate/support": "5.4.*"` みたいにしてるのをよく見るけどこれはダメ。
5.5になっただけでcomposer updateできなくなる。
プロジェクトは`5.4.*`でいいけどパッケージなら`^5.4`とか`5.*`にすれば6.0になるまではそのまま使える。
5.6で動かなくなったら？その時はその時だ。5.5で確実に使えなくなるよりはいい。

無理矢理対応するならフォークしてcomposer.json書き換えて…のいつものやり方。

## bootstrap/autoload.php
削除された結果 `artisan` `public/index.php` `phpunit.xml` などのほとんど変更することのなかったファイルが変更されてる。
ドキュメントに書かれてないのは中身は同じなので旧バージョンから使ってるならそのままでも問題ないから？
5.4での変更で`vendor/autoload.php`を読み込むだけになってた。
https://github.com/laravel/laravel/commit/8914be5fc864ebc6877be38ff3502997e0c62761

（追記）残したまま5.5にしてみたけど大丈夫だった。

## Package Discovery
5.5リリース前に書こうと思ってて間に合わなかった機能。
地味ながら便利になる。
対応してるパッケージなら`config/app.php`から削除していい。

## Request
一番気を付けたほうがいいのはここかも。
微妙に仕様変更されてる。
説明するのが難しい…。
5.4の`only()`と`intersect()`の違いを分かってる上で
https://readouble.com/laravel/5.4/ja/requests.html#retrieving-input
5.5での変更は実際に色々試してみないと。
たぶんテストでは気付かず後からあれ？？？となる。

## その他
細かい部分は使ってなければ影響はないので後はプロジェクト次第…。
  
依存パッケージの少ない小さいプロジェクトで5.5にしてみたけど問題なく終わった。

## もう少し追記
他も移行作業してたら引っ掛かった箇所。

## Exception Format
エラー時のJSONのフォーマットが変わってるのでJS側でエラーメッセージ表示してるなら修正が必要かも。
修正が面倒で5.4と同じフォーマットに戻したいならドキュメント通りに。

## Mail Fake
`Mail::assertSent`から`Mail::assertQueued`に変更する。
テスト失敗するのですぐ気付く。
こういうのが使ってなければ影響がない変更。
