---
title: "Google Cloud Platform / Always Freeを試す"
date: 2018-05-09T13:04:11+09:00
categories: ["GCP"]
draft: false
---

GCPの無料枠。インスタンス一つを無料で使えるのでどのくらい使えるか試し。
https://cloud.google.com/free/?hl=ja

細かいことは検索すれば済むのでここでは省略。

## Google Compute Engine
https://cloud.google.com/compute/pricing?hl=ja
AWSでのEC2。
LaravelかWordPressを動かせるように準備。

無料で使えるのは米国のリージョンのみ。この時点で遅いのですでに辛い。
SSHで接続していつものようにForgeでセットアップ。
Ubuntu 18.04にPHP7.2がインストールされた状態。

この前サーバーでの公開はしなかったLighthouseプロジェクトをとりあえず置く。
https://lighthouse.kawax.biz/

このくらいなら問題ないけどメモリ0.6GBなので実用にはかなり厳しい…。

インスタンス使うだけならGCPもAWSもその辺のVPSも全部同じ。
違うのは他のサービス部分。
LaravelでもWordPressでもS3をよく使うのでAWSでいいかとなる。
