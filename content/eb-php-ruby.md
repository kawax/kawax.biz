---
title: "Elastic BeanstalkでPHP/Ruby混在プロジェクトをデプロイ"
date: 2019-06-21T20:00:35+09:00
categories: ["AWS"]
draft: false
---

Elastic Beanstalk=EB

## 前提
ベースはCakePHP2プロジェクト。
CakePHP2には標準ではマイグレーション機能がない。
最初はCake用のマイグレーションプラグイン使ってたけど途中でRuby用のマイグレーションツールを使うように変更したらしい。
https://github.com/winebarrel/ridgepole
マイグレーションだけなのでRuby用でも使える。
デプロイ時にPHPとRuby両方必要になっている。

EBのプラットフォームは`64bit Amazon Linux 2018.03 v2.8.12 running PHP 5.6`
PHP7で動かないので仕方なくまだ5.6。
関係ないけど今回の目的は東京リージョンへの変更なので他は後から。

## 困ること
手動で作業するなら何も問題ない。
PHPとRuby同時にインストールできないなんてことはない。
`bundle install`してマイグレーション実行すればいい。

問題はこれをEBの設定ファイル内でやろうとするとエラー。
元からRuby2.0がインストールされてるけどridgepoleは動かないのでアンインストールしてから2.4をインストール。
`bundle install`するとインストール失敗。

```
Installing mysql2 0.5.2 with native extensions
Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
```

Ruby環境はこれが多い…。native extension gemのせいで厄介なエラーに頻繁に遭遇する。
検索すれば`ruby24-devel mysql-devel`もインストールすればいいと分かるけどEBではこれだけでは解決しなかった。

ここからは何も情報がない世界。

最終的には`PATH`環境変数がないのが原因と分かった。
解決の糸口は`env`してみたらEC2にSSHしてenvとEBのデプロイ中のenvが違ったから。
デプロイ中の環境変数が足りてない。
gemのコンパイル時になにかのコマンドが見つからずにエラー、と。

## 設定ファイル

```yaml
container_commands:
  01_install:
    command: |
      yum remove ruby20 ruby20-devel rubygems20 -y
      yum install ruby24 ruby24-devel mysql-devel -y
      gem install bundler --no-ri --no-rdoc
      /usr/local/bin/bundle install
    env:
      PATH: /sbin:/bin:/usr/sbin:/usr/bin:/opt/aws/bin:/usr/local/bin
    leader_only: true
  02_migrate:
    command: /usr/local/bin/ridgepole ...
    leader_only: true
    env:
      LC_ALL: ja_JP.UTF-8
      LANGUAGE: ja_JP:ja
```

ridgepole実行時の`LC_ALL`なんかも普通は気付けない…。

PHPプロジェクトにRuby混ぜると何倍も複雑になる。