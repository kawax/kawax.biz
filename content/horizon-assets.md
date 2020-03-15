---
title: "Laravel Horizonのアセットをデプロイ時に公開する"
date: 2020-03-15T23:24:34+09:00
categories: ["Laravel"]
draft: false
---

## 環境
- Laravel 6,7
- Horizon 3,4

## 普通にインストールすると
`horizon:install`でServiceProviderとアセット(CSSやJS)と設定ファイルが追加される。

```
php artisan horizon:install
```

使い始める時はこれでいいけどアセットはバージョンアップ時にも再公開が必要。

```
# Horizon 3
php artisan horizon:assets

# Horizon 4
php artisan horizon:publish
```

数年使った経験からは毎回公開しなくても別に壊れはしないからたまに再公開すれば十分だった。
Laravel7に合わせてHorizon4にしたら再公開が必要な変更が入っていたので今後のために「デプロイ時に自動で公開する使い方」に変える。

## 新規で使い始める場合
.gitignoreに追加。

```
/public/vendor/horizon
```

後は普通にインストール。

```
php artisan horizon:install
```

gitリポジトリには含めない。

## 既存プロジェクトを変更する場合
`/public/vendor/horizon`を削除。

.gitignoreに追加。

```
/public/vendor/horizon
```

アセットだけ再公開。

## デプロイ時
Horizonのバージョンに合わせてどちらか。

```
# Horizon 3
php artisan horizon:assets

# Horizon 4
php artisan horizon:publish
```

gitリポジトリには含まれてないのでこれで`/public/vendor/horizon`が公開される。
毎回上書き。

既存プロジェクトでは初回だけデプロイ先の本番サーバーで`git stash`が必要かもしれない。デプロイ方法次第なのでエラーが出たら各自で対応してもらうしかない。
