---
title: "Laravel Forge + Amazon Lightsail"
date: 2017-09-09T12:18:49+09:00
categories: ["Laravel", "AWS"]
draft: false
---

Laravel を動かせるサーバーの管理を楽にするサービス。

<!--more-->

https://forge.laravel.com/

実際は Laravel 以外にも使える。

## サーバーの用意

AWS の Lightsail (普通の定額制VPS) に `Ubuntu 16.04` だけインストールしたインスタンスを起動。メモリ 1GB 以上を選んだほうがいいはず。Static IP を設定しておく。ssh でログインできる所まで確認できればOK。

前にアメリカのリージョンで手動で設定して使っていた。東京リージョンへの変更は簡単にはできないので今回は Forge を使ってみる。`Ubuntu 16.04` が使える VPS ならどこのでも使えるはず。

## 登録
登録後最初の circle がどうこうの画面はスキップ。AWS の Lightsail だけど AWS も選択せずにとにかくスキップ。

画面のデザインはよく変わるようなので画像はなし。

## Custom VPS

- Name: なんでも
- Size(RAM): なんでも。2 としか入力されてないけど `2GB` の意味なはず…。
- IP Address: サーバーのIP。Lightsail で設定した Static IP
- Private IP Address (Optional): 不要
- PHP Version: 最新。7.1
- Database: 必要な DB を。MySQL(5.7)
- Database Name: なんでも。forge

`Provision As Load Balancer` はチェックせず CREATE SERVER

## Provision
Provision Command と Database Password が表示されるのでどこかに控えておく。メールも届く。

ssh でログイン後
```bash
sudo su
```

```bash
wget -O forge.sh https ...
```

root で Provision Command をコピペして実行すれば色々とダウンロードしてサーバーがセットアップされる。

Forge 側では Active Servers に追加したサーバーが表示される。

## Site
サーバーの準備ができたので次はサイトの追加。デフォルトでは `phpinfo()` だけのページが作られていて IP アドレスでアクセスしてみれば表示できる。

サーバーのページを見れば `New Site` があるのでドメインを入力して ADD SITE

- Root Domain: ドメイン
- Project Type: General PHP / Laravel
- Web Directory: /public

Active Sites に追加される。


## GitHub からデプロイ
まず GitHub の認証を済ませておく。My Account → Source Control

サイトのページに `Git Repository` と `WordPress` があるので Git を選ぶ。

- Provider: GitHub
- Repository
- Branch: master

`Install Composer Dependencies` は Laravel なのでオン。

デプロイされたら他の設定。

## Environment
サイト設定でまず Environment は Laravel の `.env` と同じ。デプロイ前に先に設定すると上書きされる。

## Queue
必要なら設定。自力で Supervisor の設定は結構大変だけど Forge なら簡単なのでキュー使ったことない人はこれを機に使ってみるといいかもしれない。

AWS の SQS を使う場合

- Connection: sqs
- Queue: `config/queue.php` の `'queue' => 'your-queue-name',` で設定したキュー名。

## SSL
LetsEncrypt で SSL 対応の設定。先にドメインで正常に表示できてないと設定失敗するので後から。

Lightsail の場合、Lightsail 側の Networking 設定で HTTPS を追加。これを忘れると接続できない。

## サイト追加
一つのサーバーで複数の Laravel アプリを動かすことは可能。
ドメイン追加すればいくつでも追加できる。インストール先は `/home/forge/{ドメイン}/` 

## Database
最初に Custom VPS で MySQL を選んだなら MySQL がメニューにある。

まだ試してないけど Add Database の Name が .env の `DB_DATABASE` になるはず。User と Password は設定しなければ Provision 時に設定されたパスワードで使える？ なんでも使える forge ユーザーを使いたくない場合は新しく作る。

## Scheduler
サーバー設定にある。`php /home/forge/default/artisan schedule:run` の `default` をドメインにして設定。

最初から `composer self-update` があるようにただの cron なので Laravel のタスクスケジュール以外でも使える。

## Travis
`Quick Deploy` をオンにすれば git push でそのままデプロイされるけど Travis CI でテストを挟みたい場合。`Quick Deploy` はオフのまま。

Travis の Environment Variables で Forge の Deployment Trigger URL を設定。

- Name: なんでも。FORGE
- Value: https://forge.laravel.com/servers/ ...

.travis.yml に追加。トリガー URL に get でも post でもリクエストがあればデプロイ開始される。

```
after_success:
   - curl -s "$FORGE";
```

## Deploy Script
`composer install` に `--no-dev` がないので追加しておく。Laravel Dusk などが原因で install でエラーになる場合は `Install Composer Dependencies` をオフにして一度デプロイした後、Deploy Script を変更して再度デプロイすれば解決。

DB 使ってないなら migrate 部分を削除かコメント。
```bash
#if [ -f artisan ]
#then
#    php artisan migrate --force
#fi
```

## Notifications
Travis でも通知できるけど Forge を使うほうが簡単かも。

## 完成
ここまでできれば GitHub への push → Travis CI でテスト → Forge がデプロイ。いつもの流れが実現できた。

## 感想
Forge はかなり楽。VPS 1台でいいような小さいサイトならこれで良さそう。

## 追加の AWS 版
Lightsail は個人用の小さいサイトだけど業務でも使えるように AWS 版も試す。EC2 にセットアップする形。

まず My Account → Server Providers で Amazon のアカウントを追加。Profile Name は識別用なのでなんでも。IAM ユーザーの Access Key と Secret Key を入力。EC2 を操作できる権限が必要なはず。

Create Server で AWS と Custom VPS が選べるようになってる。

...

EC2 と キーペアと VPC が新しく作られる。AWS で使うなら既存の VPC に入れて RDS 使ったりしたいのでこれはちょっと違う。削除機能を試して終わり。EC2 と キーペアは削除されたけど VPC は残ったまま。VPC だけ手動で削除。

EC2 使うなら普通に新しく起動して Custom VPS と同じようにセットアップすれば良さそう。 

## EC2 + GitLab(自分で運用版)
次に普通に建てた EC2(Ubuntu) に GitLab のプライベートリポジトリからのデプロイを試す。Source Control にある GitLab は gitlab.com なので別。WordPress を自分のサーバーで動かすのと .com 版の違いと同じ関係。

Install Repository の画面で Custom を選べば SSH key を追加するように書いてあるので GitLab 側のユーザー設定で SSH Keys を追加。

- Provider: Custom
- Repository: `ssh://〜 .git` の URL を指定。

エラーが出た場合は…？ 色々やってたら解決してたので原因は不明。キーのパーミッションを変えたり。最終的は EC2 内で直接 `git clone` して成功すれば forge からでも成功するはず。

```
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

clone できれば後は同じ。GitLab CI でテストしてトリガー URL からデプロイ実行。

## Bitbucket
認証しておけばGitHub と同じように使える。

## 終わり
結局 10 サイトくらい Forge 管理に移行した。Let's Encrypt がワイルドカードサブドメインに対応してないのでそれが必要なサイトだけ残してる。
