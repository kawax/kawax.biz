---
title: "GitHub ActionsでLaravelのテストを実行"
date: 2019-10-23T20:40:22+09:00
categories: ["Laravel"]
draft: false
---

## はじめに
GitHub Actionsはまだベータなのでこれからいくらでも変更される。あくまで執筆時点での情報。

ベータは申し込めば数日後には使える。

## 準備
GitHubにLaravelプロジェクトを用意してActionsを選べばすぐにLaravel用のテンプレートが出てくるのでこれでセットアップするだけ。
準備も何もない。

デフォルトではこのyamlファイルが`.github/workflows/laravel.yml`に作られる。

```yaml
name: Laravel

on: [push]

jobs:
  laravel-tests:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Copy .env
      run: php -r "file_exists('.env') || copy('.env.example', '.env');"
    - name: Install Dependencies
      run: composer install -q --no-ansi --no-interaction --no-scripts --no-suggest --no-progress --prefer-dist
    - name: Generate key
      run: php artisan key:generate
    - name: Create Database
      run: |
        mkdir -p database
        touch database/database.sqlite
    - name: Execute tests (Unit and Feature tests) via PHPUnit
      env:
        DB_CONNECTION: sqlite
        DB_DATABASE: database/database.sqlite
      run: vendor/bin/phpunit
```

SQLite使うように設定されてるので普通のLaravelの使い方してるならこれで十分。

自分でインメモリSQLite使うようにしてるならSQLite部分は削除してもいい。

ドキュメント見て好きなように変更すればいい。
https://help.github.com/ja/github/automating-your-workflow-with-github-actions/about-github-actions

## cron
今の所は分まで指定して自由に使えるっぽい。タイムゾーンはUTCで指定。

```yaml
on:
  schedule:
  - cron: 0 3 * * *
```

ただし **「サーバーレスコンピューティングには使うな」** と禁止してるのでいずれ制限されそう。
