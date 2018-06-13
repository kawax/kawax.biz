---
title: "ForgeからEC2インスタンスを起動する"
date: 2018-05-25T11:40:30+09:00
categories: ["Forge"]
draft: false
---

以前はVPCを指定できなかったので使いにくかったけど最近指定できるようになったので普通に使えるようになった。

## アクセスキーの設定
AWSのIAMで`AmazonEC2FullAccess`権限を持ったユーザーを作りアクセスキーを取得。

Forge側のMy Account→Server ProvidersでAmazonを追加。

## Create Server

<img src="/img/forge-ec2/forge-ec2.png">

サーバー追加画面で選べるようになる。
AWS側にログインせずにEC2を起動できる。
最近は複数のAWSアカウント扱うようになってきたので楽かも。

Custom VPSではセットアップ作業を手動で行っていたけどこの方法ならPHPやnginxのインストールまで自動。

## Security Group
defaultが使われるのでforgeからのSSHを許可してないとサーバーのセットアップができない。

## SSH Key
これで起動したインスタンスは新しいキーペアが作られてるけど入手手段はなさそうなので別のキーでSSH接続できるようにする。

```
ssh-keygen -y -f forge.pem  
```

pemから公開鍵を作りForgeのSSH Keysに設定。

## SSH config
SSH接続時にはこのpemとユーザー名にforgeを指定する。

```
Host ec2
  HostName xxx
  User forge
  IdentityFile ~/.ssh/forge.pem
  Port 22
```

forgeユーザーでログインしてるのでsudo時にはパスワードが必要になる。
サーバー追加時にメールが来てるはずなのでしっかりメモしておく。
