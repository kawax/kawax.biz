---
title: "Predisが開発再開したのでLaravelでの非推奨も解除された"
date: 2020-08-21T17:34:05+09:00
categories: ["Laravel"]
draft: false
---

「composerでインストールするpredis」と「peclでインストールするPhpRedis」では手軽さが違うので良かった。
https://github.com/predis/predis
https://github.com/laravel/framework/pull/33872

パフォーマンスはPhpRedisのほうがいいけどそこまで求めない場合はpredisでいい。

predisに戻そう。