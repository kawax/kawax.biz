---
title: "Laravel/PHP用パッケージ開発時のコツ"
date: 2020-02-26T12:41:53+09:00
categories: ["Laravel"]
draft: false
---

もしかして、Laravel用パッケージの開発に関してはいつの間にか自分が日本で一番詳しくなってるのでは…。
パッケージ作って何年もメンテナンス続けてる人は見た記憶がない。
実際は日本だけでなく海外にもそんなに多くない。Laravelのバージョンアップに対応できずに捨てたパッケージは大量にある。

## まずは素のPHPでも使えるように作る
わざわざLaravel専用にする理由はないので素のPHPや他のフレームワークでも使えるように作る。
その上でLaravelなら簡単に使えるようにFacadeなどを整備。

Laravelの機能を拡張するパッケージなら最初から専用でいい。

composer使うのは書くまでもない当たり前の前提。

## サービスコンテナやサービスプロバイダーの理解が必須
普通にLaravel使うだけなら不要だけどパッケージ作るなら必須。
というよりパッケージ作ってると勝手に身に付く。

## Laravelのバージョンアップは常に追いかける
Laravelに合わせてパッケージのメンテナンスも必要。
パッケージ作るなら半年毎のメジャーバージョンアップなんて狂気に付いていく覚悟が必要。

## 旧バージョンのサポートは容赦なく切る
PHPもLaravelもサポート期限ははっきり決まってるので旧バージョンのサポートは切っていい。
きちんとメンテナンスされてるかどうかはここで決まる。
新機能なんか追加しなくていい。
サポート中の現行バージョンと次期バージョンで問題ないかの確認。やるべきことはこれだけ。

## Laravel公式の真似をする
機能の使い方やコードの書き方などとにかくすべて公式を参考にする。非公式な情報は一切見てはいけない、全部間違ってると思っていい。
意外と非公式なものを公式と思い込んでる人は多い。GitHubにあるものだけが公式。
https://github.com/laravel

## composer.jsonのバージョン指定
ここは各自の方針で決める所。
Laravel用パッケージなら大抵requireに`illuminate/support`を指定する。
これでLaravelの対応バージョンを指定するけど
```
"illuminate/support": "^5.8||^6.0||^7.0",
```
半年毎に書き換え作業したくないなら`*`で指定する方法もある。
```
"illuminate/support": "*",
```
ただし`*`を使うのはあまり良くないのでどうするかは各自で決める。

## TravisでPHP8.0(開発版)のテスト
たぶん2020年中にしか役に立たない。

Travisは`nightly`でその時点での最新の開発版が使えるけどPHP8.0はまだ大半のパッケージが対応してないのでそのままインストールしようとしてもできない。`"php": "^7.2||^8.0",`としてくれるのを待つしかない。
composerのオプションで`--ignore-platform-reqs`を付ければ無視できるのでnightlyのみ付けるように`.travis.yml`を調整。

```yaml
sudo: false
language: php

jobs:
  include:
    - php: 7.2
    - php: 7.3
    - php: 7.4
    - php: nightly
      env: COMPOSER_OPTION=--ignore-platform-reqs

before_script:
  - travis_retry composer install --no-interaction --no-suggest $COMPOSER_OPTION

script:
  - vendor/bin/phpunit
```

8.0リリース後もまだ対応してなさそうなので8.1(開発版)でテストするためにおそらくこうなる。

```yaml
jobs:
  include:
    - php: 7.2
    - php: 7.3
    - php: 7.4
    - php: 8.0
      env: COMPOSER_OPTION=--ignore-platform-reqs
    - php: nightly
      env: COMPOSER_OPTION=--ignore-platform-reqs
```