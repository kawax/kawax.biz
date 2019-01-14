---
title: "Yet Another Packagist Mirror"
date: 2019-01-12T19:30:43+09:00
categories: ["Packagist"]
draft: false
---

https://packagist.jp/ がまれに止まってるので自分でも作った。この年末年始も2週間くらい止まったままだったような。

https://packagist.kawax.biz/

https://github.com/kawax/packagist-bot

## S3+CloudFront
CloudFrontなので日本以外のどこからでも速いはず。
公式がCloudFront使ってくれればこんなミラーなんか不要なんだけど…。

## Discordへ通知
jpで困るのは止まってることに気付かないことだったので更新時には毎回通知してる。
これですぐに気付く。

## クローラー
jpのクローラー見てもよく分からなかったのでクローラーから作り直し。
中身はLaravel系。
最近はartisanコマンドだけ使うことが多い。
Laravelそのままではなく一部だけ使ってるのでLaravel系と言ってる。
プロジェクト増えすぎてそのまま使ってたらバージョンアップ作業で死ぬ。
そもそも今回のミラー作ったのも先にcomposer update自動化→jpが止まってて困るの流れ。
ローカルでupdateするならさっと無効化すればいいけどサーバー上で自動化してる場合無効化しにいく手間が面倒だった。

## 動かし方
本当は個人で運用するものではないのでどこかの会社にやって欲しい…。
実際動かすとなるとcronで回すだけではだめなので若干ややこしい。
packagistのファイルが20万以上あるので今から始めるには最初の準備が大変。

動かし方は必要な人がいたら書くことにする。

## 続き
- Docker用意したので動かし方の説明はしやすくなった。
- Dockerでのcronは色々厄介。[Laravel News](https://laravel-news.com/laravel-scheduler-queue-docker)でさえsleep 60で実行してるけど他にないんだろうか。別のイメージならcrond -fで動いたけど。
- パッケージが削除されてもミラー側では気付けない。実用上は問題ないので無視。年に1回くらいは全部削除して取得し直してもいいかもしれない。
- MacとLinux(Ubuntu18.04)でduコマンドのオプションが違う。
- ファイル数のカウントはPHPのStorage::allFiles()より`find . -type f -name "*.json" | wc -l`のほうが速い。Docker内で実行すると遅い。
