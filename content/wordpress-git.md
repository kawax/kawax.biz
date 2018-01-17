---
title: "そろそろ WordPress でも git や composer が当たり前になって欲しい"
date: 2017-12-06T11:33:45+09:00
categories: ["WordPress"]
draft: false
---

普段 Laravel などで git や composer を使ってると WordPress はもうひたすらにつらい。
テーマ変更してFTPでアップとかすると何この20世紀…。
それしかできない人に合わせて許してたけど改ざん被害が増えてきて初心者に任せるのはいい加減我慢の限界が来たので全部やり方変えることにした。

全部書くと長いのでこういう方針でやるということだけ紹介。

## 環境
- AWS。AWS使ってないレベルの人にはそもそも関係のない話。
- GitLab。comではなく自分で運営版。GitHubやBitbucketでもいい。
- デプロイやサーバー管理に Laravel Forge。https://forge.laravel.com 別になんでもいい。
- PHP7.1→公開前に7.2が出てたのでアップデート。Forge ならワンクリック。
- 200とか300とか大量のWP。少ないならここまでやる必要はないかも。

## 使うもの
https://github.com/roots/bedrock

gitとcomposer使ってる人ならこれ見れば分かる。

## 課題
サーバー1台に全部入れてWPを動かしてるとサーバー上にしか最新のデータがない状態。
複数人でテーマ編集するとあっという間に破綻する。
テーマだけ git で管理したこともあったけど結局は使えない人がいると勝手に FTP でアップするようになって終わる。
なので今回は全部 git 管理。FTP 禁止。git からのデプロイのみ。

WPの各構成要素をどう扱うかが課題。

- WP本体
- テーマ
- プラグイン
- メディアファイル
- DB

## WP本体
composerで管理。
bedrock でプロジェクトを作れば `johnpbloch/wordpress` があるので特に何もない。
そのままでもいいけどバージョン指定を `*` に変更しておくと楽かも。

```json
    "johnpbloch/wordpress": "*",
```

デフォルトでは `4.9.0` とか固定してるけどこれだと毎回手動で書き換えが面倒。扱うWPが一つならいいけど大量にあるとそんなことやってられない。

### ついでに config
デフォルトは自動更新をオフにしてるのでオンにする。
cronもWP標準のものを使う。
これも大量のWPだからこうする。バージョンアップの度に全部composer updateとか一つ一つcronの設定とかはしたくない。
なんらかのプラグインを使ってWPもプラグインも全部自動更新。

/config/application.php

```
define('AUTOMATIC_UPDATER_DISABLED', false);
define('DISABLE_WP_CRON', env('DISABLE_WP_CRON') ?: false);
```

production.php

```
define('DISALLOW_FILE_MODS', false);
```

ただし、WPデフォルトでは `.git` が存在すると自動更新されない。  
https://wpdocs.osdn.jp/%E8%87%AA%E5%8B%95%E3%83%90%E3%83%83%E3%82%AF%E3%82%B0%E3%83%A9%E3%82%A6%E3%83%B3%E3%83%89%E6%9B%B4%E6%96%B0%E3%81%AE%E8%A8%AD%E5%AE%9A  
functions.php に1行書けば解決するとはいえ大量のWPでは面倒なのでプラグインにした。これならcomposer.jsonにコピペで済む。  
https://packagist.org/packages/revolution/enable-automatic-updates-vcs  
WPの公式には登録せずに使える。登録するとバージョンアップの度に対応が必要で結構面倒。svnとかPSR-2とは違うコーディング規約とかなにもかもが違って本当につらい。あくまで公式に登録する場合に守る規約なので登録しないなら無視してPSR-2にすればいい。

cronも動かないようなら仕方ないので wp-cliで代わりに実行。

```
wp cron event run --due-now
```

Forge なら

```
cd /home/forge/example.com/; /usr/local/bin/wp cron event run --due-now
```

（追記）その後の様子を見る限り wp-cron.php だけで問題なく動作している。

## テーマ
gitリポジトリ内に全部入れる。

## プラグイン
wpackagistのおかげでWPプラグインもcomposerで管理できる。
https://wpackagist.org/

これもバージョン指定は `*`

たまにファイルを作ったりwp-config.phpを書き換えたりするプラグインがあるけどそういうのは個別になんとか対応する。
wp-config.php の書き換えは事前に書き換えておけばいい。
どうしても対応できないプラグインは使用を諦める。

## メディアファイル
これが一番問題だけどS3にもアップするプラグインを使えばいい。  
Argiope amoena  
https://ja.wordpress.org/plugins/argiope-amoena/  
Nephila clavata  
https://ja.wordpress.org/plugins/nephila-clavata/  

その他各自の好きなものを。ただしS3にファイルがあってサーバー上にない場合にS3からダウンロードする機能がないと使いにくいはず。

EFSさえ東京リージョンに来ればここも簡単になるはずだけど中々来ない…。  
https://aws.amazon.com/jp/efs/

サーバー1台だけで使うならS3なしでもいい。
前に Elastic Beanstalk 使った時はデプロイの度にメディアファイル消えるのでS3使うしかなかった。
EC2とかどこかのVPS1台だけで増やすつもりがないならそのままサーバー内だけでも問題はないはず。

## DB
RDS使うだけ。元々RDSは使ってたので変更なし。

## 全体の構成
それぞれの扱い方が決まったので全体的な構成。

### Route53→ELB(ALB)→EC2
https://example.com

メディアファイルとDB以外はGitLab→Laravel Forgeを経由してEC2へデプロイ。EC2増やして複数台構成にもできる。CloudFront使いたいならRoute53とALBの間に入れて使える。

### Route53→CloudFront→S3
https://image.example.com

メディアファイルはプラグインがS3へアップ。httpsのためCloudFrontを通して表示。
ただし、大量のWPで全部CloudFront設定するのは面倒すぎるのでS3だけでもいいかもしれない。
`https://s3-ap-northeast-1.amazonaws.com/{S3バケット}/...` のURLならS3だけでもhttpsにできる。

### RDS
特になし。

## HTTPS
ACMで証明書作ればいいだけだけど
EC2用は東京リージョンで作ってALBで設定。
S3用はバージニアリージョンで作ってCloudFrontで設定という違いがあるので注意。

WPにもプラグインが必要かもしれない。
https://ja.wordpress.org/plugins/really-simple-ssl/

この辺の細かい所は実際にやってみるしかない。

## デプロイ
ローカルで開発や動作確認などしたらgit push。
GitLab CIがForgeのDeployment Trigger URLをキック。
.gitlab-ci.yml でこんな

```
curl -s "https://forge.laravel.com/..."
```

後はForgeがデプロイ。

ForgeのDeploy Scriptでwp-cli使えばさらに処理を挟めるはず。

Forge を使わない場合はgit pullとcomposer installすればいいだけなので各自のやり方で。

## セキュリティの副産物
WPへの攻撃は大体 `wp-login.php` や `wp-content` 以下決め打ちなのでbedrockの構造なら攻撃が激減する。
ログインURLはさらにプラグインで変更してるので不正ログイン試行さえほとんどなくなった。

ALBならAWS WAFが直接使えるので最近追加されたマネージドルールが使える。（古いELBではCloudFrontも必要だった。）
https://dev.classmethod.jp/cloud/aws/aws-reinvent2017-waf-managed-rule-try/

## 終わり
書ききれなかった部分でも色々あるはずだけどとりあえずこんな感じでやれば「いつもの」ができる。
ローカルサーバーにHomestead、js/cssビルドにMixでガチガチにLaravel流にしてるけどテーマにBlade使うのはさすがに他の人が触れないのでやめた。

重要なのは独自すぎる仕様にしないこと。全部外部に使い方のドキュメントがある。

[WordPress Advent Calendar 2017 - Qiita](https://qiita.com/advent-calendar/2017/wordpress)
