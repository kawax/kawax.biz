---
title: "Laravel Forge 複数サーバー構成にする"
date: 2018-02-26T21:00:58+09:00
categories: ["Forge"]
draft: false
---

## 変更前の環境
サーバー：server-1
サイト：example.com

ここにserver-2を追加する。

AWSのEC2+RDSだけどVPSで1台にDBも入れてる環境でもいいはず。

## 追加サーバーの準備
起動してForgeに追加してセットアップ、とForgeの通常通りの手順でサーバー設定。
同じドメインでサイトも追加する。

サーバー2つとサイト2つの状態。

- example.com(server-1)
- example.com(server-2) まだ表示できない。

## SSL
Let's Encryptなら、`Clone Certificate`でserver-1から持ってくるだけで使える。

## ネットワーク設定
DB、Redis、memcachedはserver-1のものを使う。

server-2のNetwork→Server Networkに`server-1`があるので選択してUpdate Network。

> Below is a list of all of the other servers this server may access. In other words, this server's network! This makes it a breeze to use a connected server as a separate database, cache, or queue box.

これで具体的に何をやっているかというと`ufw`で必要なポートを許可している。
server-2のIPのみ許可。

Network設定ページ下部のActive Firewall Rulesに出てこないけどやってることは同じ。

これでserver-2からでもDBやRedisのHOSTにserver-1のIPを指定すれば接続できるようになる。

AWSの場合はセキュリティグループでも必要なポートを許可する。Redisなら`6379`、memcachedなら`11211`など。

## Environment
サーバー設定で接続できるようになったのでserver-2のサイト側でEnvironmentの設定を変更。  
server-1のIPにする。

```
DB_HOST=
REDIS_HOST=
MEMCACHED_HOST=
```

## DNS
あとは実際の表示時にサーバー2つに振り分けられるようにする。

AWSでELB使ってるならELBにserver-2追加。  
ELB使わずDNSで直接IPを指定してるならserver-2のIPも追加してラウンドロビン。

問題なく表示できるか確認。  
失敗する場合はポートの許可を確認。

## その他
Laravelの場合はタスクスケジュールやキュー、Horizonなどをどう設定するか考える必要がある。  
タスクスケジュールは一つのサーバーでのみ設定するか、Laravel5.6で追加された`onOneServer()`を使う。  
Horizonは同じRedisなら両方で動かしても大丈夫。

## デプロイ
Travis経由でTrigger URLをキックしてるならserver-2の分も追加。  
Quick deployを使ってるなら有効化。  
サーバーが増えてもpush1回で全部にデプロイできるので手間は変わらない。  
数台程度の規模ならこれで十分。  
もっと大規模なら違う方法にするけど。
