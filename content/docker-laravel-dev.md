---
title: "DockerでHomestead並のLaravel開発環境を作る"
date: 2019-01-20T18:01:36+09:00
categories: ["Laravel"]
draft: false
---

artisanコマンドしか使わないプロジェクトにHomesteadは過剰なので直接`php artisan ...`を実行してるけどartisanのみでもRedisやキューが必要な時もある。こういう時がDockerの使い所。  
packagistがまさにそんな例。  
https://github.com/kawax/packagist-bot

これで常時Docker起動するようになったので普通のLaravel開発環境もDockerで作ってみた。
https://github.com/kawax/docker-laravel-dev

- プロジェクトごとに分離。
- hostsの設定を楽に。
- cron、キュー、Horizon、ブロードキャストまで使えるように。

検索して出てくる日本語でのDockerでLaravel開発環境作ってみた記事はどれもこれもPHPとMySQL程度しか使ってなくて何一つ参考にならなかった。

今回は最終的に使わなかったけどBonjour使うためにDockerのネットワーク周りを色々調べてたら奥が深すぎてLaravelでアプリ作るなんて程度は誰でもできる簡単なことだと改めて思った。Laravelの楽さに慣れすぎてた。

macvlanドライバー使えばどうにかできそうだったけど途中で断念。
https://docs.docker.com/network/macvlan/
