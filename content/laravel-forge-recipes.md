---
title: "[WIP]Laravel Forge Recipes"
date: 2018-01-11T00:54:32+09:00
categories: ["Laravel"]
draft: false
---

Laravel Forge でデプロイとサーバー管理をしていると直接SSHする必要はほとんどなくなる。  
直接SSHせずにサーバー上でなにかしたい時のために Forge には Recipe という形で用意されている。  
デフォルトでは何もないので自分で作る。一度作れば他のサーバーでも使い回せるし複数サーバーに対して同時実行もできる。実行の結果はメールで送られてくる。  
作った Recipe をここに残しておく。  

<img src="/img/laravel-forge-recipes/laravel-forge-recipes.png">

## Forge の IP
直接しないなら必要ないということでSSHのポートは閉じて Forge だけ許可している。  

```
45.55.52.11
104.236.229.125
104.236.3.83
```

https://forge.besnappy.com/laravel-forge#servers-5454  
一応これが公式情報のはずだけど微妙に違う情報もあるのでAWSのセキュリティグループでは
`45.55.0.0/16` `104.236.0.0/16` で許可。  
もちろん一時的にSSHしたい時は自分のIPを許可すればいい。

### 2018-01-31
変更の案内が出ていた。

```
Forge IP Addresses: We are upgrading our servers soon! If you use IP whitelisting, please add our new addresses: 159.203.161.246 and 159.203.163.240.
```

## Amazon Time Sync Service
時刻同期を Amazon Time Sync Service を使うように設定。ついでにタイムゾーンを `Asia/Tokyo` に。

対象 : AWSで動かしてるUbuntu 16.04

User : root

```
sed -i -e 's/#NTP=/NTP=169.254.169.123/' /etc/systemd/timesyncd.conf
service systemd-timesyncd stop
service systemd-timesyncd start

timedatectl set-timezone Asia/Tokyo
timedatectl
```

## Install wp-cli
wp-cli をインストール。

User : root

```bash
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
mv wp-cli.phar /usr/local/bin/wp

wp --info
```

## Install Mackerel

Ubuntu 16.04用のコマンドをそのままコピペすればいい。

User : root

```bash
wget -q -O - https://mackerel.io/file/script/setup-all-apt-v2.sh | MACKEREL_APIKEY='' sh
```

## apt upgrade

たまに更新の途中で確認画面が出るのでRecipeからの実行は避けたほうがいいかもしれない。lockファイルの削除など面倒なことになる。

User : root

```bash
apt update
apt upgrade -y
```
