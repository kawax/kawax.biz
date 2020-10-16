---
title: "composerパッケージには.gitattributesファイルを含めるべき"
date: 2020-10-16T21:07:32+09:00
categories: ["Laravel"]
draft: false
---

gitでは属性を設定するみたいな話だけど
- https://git-scm.com/docs/gitattributes
- https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA-Git-%E3%81%AE%E5%B1%9E%E6%80%A7

composerパッケージとGitHubの文脈では
「composer install時にダウンロードしないファイルを.gitattributesで指定する」という意味。

## composer install時の動作

```
composer install
```
何も指定しなければ`--prefer-dist`オプション付きと同じ。
Zipファイルをダウンロードしてインストールするのがデフォルトの動作。

GitHub側の「Download ZIP」と同じ。

.gitattributesで`export-ignore`を指定するとこのZipファイルに含まれなくなる。
インストール時に不要なファイルをダウンロードしなくていいように指定する。

## ライブラリやフレームワークの.gitattributesを確認

- https://github.com/guzzle/guzzle/blob/master/.gitattributes
- https://github.com/briannesbitt/Carbon/blob/master/.gitattributes
- https://github.com/laravel/framework/blob/8.x/.gitattributes
- https://github.com/cakephp/cakephp/blob/master/.gitattributes
- https://github.com/symfony/symfony/blob/5.x/src/Symfony/Component/VarDumper/.gitattributes

Symfonyは各Componentには含まれている。

## 残すファイル
動作に関係ないファイルは消していいけどREADMEやLICENCEは残す。

## tests
元々不要なファイルを消すか消さないかだけなので動作への影響はなさそうに見えるけど影響することもある。
パッケージのテストの名前空間を`Tests`みたいにしていた場合、重複して困ることがある。

```
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    },
```

`tests`ディレクトリも削除対象。

## composer.lockの扱い
本来ライブラリではcomposer.lockはリポジトリに含めなくていい。
色々な開発の都合でどうしても含めたい場合は.gitattributesでだけ指定してcomposer install時には含まないようにすれば一番良い形で解決。
