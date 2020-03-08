---
title: "自動でcomposer updateしてPull Requestを作るGitHub Actionsを作った"
date: 2020-03-08T15:37:17+09:00
categories: ["GitHub Actions"]
draft: false
---

composer使ってるプロジェクトなら何でも使えるはず。
https://github.com/kawax/composer-update-action
https://github.com/marketplace/actions/composer-update-action

PHP7.4が下限なので7.4ではインストールもできないプロジェクトではcomposer.jsonでconfig.platformの設定が必要かも。

プライベートリポジトリでも使える。プランごとに月に何分までの制限があるけど。

## 使い方
`.github/workflows/update.yml`を作るだけ。

UTCの0時は日本時間で9時。好きなように変更する。

```yml
name: composer update

on:
  schedule:
    - cron: '0 0 * * *' #UTC

jobs:
  composer_update_job:
    runs-on: ubuntu-latest
    name: composer update
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: composer update action step
        uses: kawax/composer-update-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## GITHUB_TOKENは自動的に使える
ymlにこれさえ書けばいい。

```yml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

パーソナルアクセストークンの作成は不要。Secretsは使わなくていい。

## composer.jsonがルートにない場合
`COMPOSER_PATH`でcomposer.jsonがあるサブディレクトリを指定。

```yml
        env:
          COMPOSER_PATH: /subdir
```

## composer.jsonが2つある場合

2回に分けて実行。

```yml
      - name: composer update action step
        uses: kawax/composer-update-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: two
        uses: kawax/composer-update-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          COMPOSER_PATH: /subdir
```

## コミットユーザー

```yml
        env:
          GIT_NAME: cu
          GIT_EMAIL: cu@composer-update
```

Secretsを使ってもいい。

```yml
        env:
          GIT_NAME: cu
          GIT_EMAIL: ${{ secrets.GIT_EMAIL }}
```

## GitHub Actionsのバージョン
`@v1`で指定してる例が多いからcomposerの`^1.0`と同じかと思ったけど違った。`v1`というタグをわざわざ付け替えてる。そんな面倒なことしたくないので`v1`はブランチで使用することにした。`@v1`ならv1ブランチから使う。タグは普通に`v1.0.0`。

```yml
steps:    
  - uses: actions/setup-node@74bc508 # Reference a specific commit
  - uses: actions/setup-node@v1.0    # Reference the major version of a release   
  - uses: actions/setup-node@master  # Reference a branch
```
https://help.github.com/ja/actions/building-actions/about-actions#versioning-your-action

## 終わり
自動composer update自体は1年以上前から毎日動かしてたけど結構負荷が高いので自分のサーバーで動かすと色々辛い。
GitHub Actionsに移したのでかなり楽になった。
