---
title: "MediaWiki を Push to Deploy"
date: 2018-10-25T17:25:31+09:00
categories: ["MediaWiki"]
draft: false
---

Wikiシステムを探してたけど結局MediaWikiが一番良さそう。最低でもgitとcomposer使ってないと選択肢にも入らない。  
- https://github.com/wikimedia/mediawiki

DokuWikiやPukiWikiはデータベース不要を売りにしてるけど逆に使いにくい。レンタルサーバーにFTPでアップしてた頃ならならともかくPush to Deployが当たり前な時代には辛い。遥か昔はPukiWiki使ってた記憶がある。アプリのヘルプページにはWikiがちょうどいい。
- https://github.com/splitbrain/dokuwiki
- https://pukiwiki.osdn.jp/

## 環境
- AWS。EC2+RDS+S3
- MediaWiki 1.31.1時点
- Forge。https://forge.laravel.com/　LaravelやWordPressと全く同じように管理・運用できるようにするのが目的。
- Ubuntu16.04/PHP7.2/nginx/memcached

## ローカルで動かす
参考ページ：https://www.mediawiki.org/wiki/Download_from_Git/ja

```
git clone https://github.com/wikimedia/mediawiki.git mediawiki
```
全部クローンするとかなりのサイズなので`--branch`や`--depth=1`付けて減らしたほうがいいかもしれない。

ここで自分用の別ブランチを作っておく。

```
cd mediawiki
git submodule update --init
```
サブモジュールはちょっと分かりにくいので削除して通常通りに扱ってもいいかもしれない。composerのvendorは削除した。普通にcomposer install/update。

ローカルサーバーはいつものHomestead。Homestead.yamlの`public`を削除。ドキュメントルート直下にindex.phpがある。
```
sites:
    -
        map: mediawiki.test
        to: /home/vagrant/code
```

この時点では設定ファイルはないのでまずサーバーを起動してブラウザで表示。基本的な設定を行い最後に`LocalSettings.php`がダウンロードされる。`LocalSettings.php`はドキュメントルートに置く。この`LocalSettings.php`はローカル用。gitには含めない。本番用の`LocalSettings.php`は後で再度設定して生成。
https://www.mediawiki.org/wiki/Manual:Configuring_MediaWiki/ja

動作確認できたらローカルでの作業は終わり。カスタマイズする箇所はほとんどないので拡張機能の追加やバージョンアップ時以外はローカルでの作業はたぶんない。

## S3対応
アップロードファイルをS3に置くための拡張機能  
https://www.mediawiki.org/wiki/Extension:AWS

git cloneするように書いてあるけどサブモジュール使ってる場合はこれではダメなのでサブモジュールで追加。デプロイ時に`composer install`

https://github.com/edwardspec/mediawiki-aws-s3
Tokyoリージョンの場合デフォルトの`$1.s3.amazonaws.com`では表示されないので`LocalSettings.php`に設定追加。
```
$wgAWSRegion = 'ap-northeast-1';
$wgAWSBucketDomain = 's3-ap-northeast-1.amazonaws.com/$1';
```

## GitHubからForge
この辺りはLaravel等と同じなので省略。Forgeに追加時の最初はcomposer installしないほうがいいかも。

ForgeのDeploy Script。今後変更する可能性は高い。
```
cd /home/forge/wiki
git pull origin wiki
composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader
git submodule update --init
cd /home/forge/wiki/extensions/AWS
composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader

echo "" | sudo -S service php7.2-fpm reload
```

## 本番のLocalSettings.php
再度設定した後サーバー上にLocalSettings.phpを作る。FTPは使ってないので新規ファイル作ってコピペ。
今後も設定変更する時はサーバーで直接。.envの仕組みとかないのでこうするしかなさそう。

細かい設定項目は検索すればいくらでも出てくる。

## 短いURL
デフォルトでは`https://~/index.php?title=メインページ`で表示される。これを`https://~/wiki/メインページ`にする。

https://www.mediawiki.org/wiki/Manual:Short_URL/ja

いろいろ書いてあるし検索もしたけどこれ使うのが一番早かった。
https://shorturls.redwerks.org/

## 終わり
まだ試しに稼働させてみただけなので問題なく運用できるかやバージョンアップの手間の確認はこれから。
