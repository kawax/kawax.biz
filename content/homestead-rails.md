---
title: "Railsの開発環境にもHomesteadが使える"
date: 2019-01-29T12:58:38+09:00
categories: ["Rails"]
draft: false
---

Laravel用のHomesteadだけど実はRubyとRailsもインストール済なのでRails用にも使える。

## 前提
- ホスト：Mac
- 普段はLaravel使ってる人向けなのでPHP、composer、Vagrantはホスト側にインストール済。
- Rubyしか使ってない人はcomposerが使える状態まで準備。
- Ruby/Railsの環境はHomestead内だけに構築。経験上ホスト側でやると面倒なことになる。
- 基本的には全部Homestead内で作業。開発とブラウザはホスト側。
- Rails対応はまだ作業途中なので将来のバージョンでは簡単な設定するだけで使えるようになるかもしれない。その場合は公式ドキュメントを参照。この記事はHomestead8.0時点での情報。

## Homesteadとは
Laravel公式のVagrant box。
https://laravel.com/docs/master/homestead
https://readouble.com/laravel/5.7/ja/homestead.html

Railsの開発環境はこれさえ使えばいいというものがなくて初心者はホスト側で`rails server`してることが多いと思う。
少し進んでもぐぐって誰かが作ったVagrantやDockerを使う。
自分で一からVagrant内でRubyインストールしてるような信じられない記事を大量に見かける。
たかが開発環境の構築にそんな無駄な苦労はしなくていい。
すでに用意されてるものを使えばいいけど個人が作ったものは結局メンテされてない。

HomesteadならLaravel公式で信用できてきちんとメンテされている。
node.js・MySQL・PostgreSQL・Redis・memcached・supervisor…。Railsで使うものも大体揃ってる。

## 新規作成
ホスト側。
まず空のディレクトリを作る。
```
mkdir my-project && cd $_
```
composerでHomesteadだけインストール。空のディレクトリ内でいきなりこれでいい。
```
composer require laravel/homestead --dev
```

以下のコマンドでHomestead.ymlとVagrantfileなどが作られる。
この辺りはプロジェクトごとにインストールの手順。
https://readouble.com/laravel/5.7/ja/homestead.html#per-project-installation

```
# Mac / Linux
php vendor/bin/homestead make
# Windows
vendor\bin\homestead make
```

Homestead.ymlだけ編集。Vagrantfileは触らなくていい。
ipは各自の環境に合わせて。backupでvagrant destroy時に自動でDBをバックアップ。
portsはrails serverの3000。homestead.testとかのドメイン名はRailsでは使わないので無視していい。
```
ip: 192.168.10.10

backup: true

ports:
    - send: 3000
      to: 3000
```

Vagrantの起動。初回はboxのダウンロードにそこそこ時間がかかる。
```
vagrant up
vagrant ssh

cd code/
```
ホストのプロジェクトディレクトリとvagrant内の`/home/vagrant/code`が同じ。Homestead.ymlで設定してるだけなのでcdが面倒なら変更すればいい。

以降はvagrant内での操作。

## Railsプロジェクトの作成
ここは普通。`rails`はグローバルにインストール済なので`bundle install`もグローバルでもいいとは思う。好みで。
```
rails new . -B -d mysql --skip-turbolinks --skip-test
bundle install --path vendor/bundle
```
MySQL使う場合はエラーになるので追加インストール。ここはnative extensionがあるRailsならではの苦労しそうな所。
```
sudo apt-get update
sudo apt-get install libmysqlclient-dev --fix-missing
```
再開
```
bundle install --path vendor/bundle
```

データベースの設定。

- user: homestead
- password: secret
- database: homestead
- host: localhost
- port: 3306

なので合わせる。Sequel Proなどホスト側から接続するにはhost:127.0.0.1とport:33060

config/database.yml
```
default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: homestead
  password: secret
  socket: /var/run/mysqld/mysqld.sock

development:
  <<: *default
  database: homestead
```

これでRailsが起動できる。
```
rails server
```

ホスト側で http://localhost:3000/ が表示できれば成功。

後は普通のRails。

## 日々の作業
```
vagrant up
vagrant ssh
cd code/
rails s
Ctrl-C
exit
vagrant halt
```

自動で`rails s`起動することもできるけど開発中ならvagrant内で作業してるからこれで十分だろう。

以上。

https://github.com/kawax/homestead-rails
