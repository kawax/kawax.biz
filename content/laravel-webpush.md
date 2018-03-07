---
title: "Laravel Homestead 自己証明書でSSLを有効にする（Webプッシュ通知）"
date: 2018-03-07T20:16:58+09:00
categories: ["Laravel"]
draft: false
---

（Qiita/Kobitoからサルベージ）

最近の Chrome は自己証明書では警告が出るようになっている。ローカルでの開発時には気にせずそのまま使ってもいいけど一部機能でちゃんと対応する必要があったので対応した。

## 環境
- Laravel 5.5
- Homestead 6.5.0。boxは4.0.0。
- Mac。Windowsでも証明書の登録方法が違うくらいのはず。

（サルベージ版）Homesteadのバージョンが上がったくらいでやり方は変更なし。

## サンプルに使うデモ
Web Push がまさに警告が出てる状態では使えない機能
https://github.com/cretueusebiu/laravel-web-push-demo

## 準備
デモを clone。
README 通りに VAPID の設定まで進める。
途中で PHP に `gmp` が必要と出るかもしれないけどその辺りはなんとかする。

Homestead をプロジェクトにインストール

```
composer require laravel/homestead --dev
php vendor/bin/homestead make
```

Homestead.yaml はこんな

```
folders:
    -
        map: /Users/{user}/Sites/laravel-web-push-demo
        to: /home/vagrant/code
sites:
    -
        map: homestead.test
        to: /home/vagrant/code/public
```

hosts 設定して vagrant up。

## 失敗することの確認
https://homestead.test/ を見ると警告が出てるはず。
今は無視してユーザー登録。
https://homestead.test/home
`Enable Push Notifications` ボタンが有効にならないのが確認できるはず。
`Send Notification` はデータベース通知のみ動く。

Chrome の開発ツールでもエラーが出てる。

## 有効化
Homestead では証明書自体は vagrant 内で作られてるのでホスト側に持ってくるだけ。

```
vagrant ssh
cd /etc/nginx/ssl/
ll
cp homestead.test.crt /home/vagrant/code/
```

`homestead.test.crt` のファイル名は Homestead.yaml で設定したドメインなので適切なファイルを選ぶ。

ホスト側のプロジェクトフォルダにもコピーされるので Mac ならダブルクリックでキーチェインアクセスが起動して追加される。
`homestead.test` の証明書を選んで「常に信頼」に変更して保存。
これだけで終わり。

コピーした `homestead.test.crt` は不要なので削除を忘れないように。

## 成功することの確認
https://homestead.test/home をリロードか一度閉じて開き直す。
緑の「保護された通信」になってることを確認。

`Enable Push Notifications` で Web Push の許可画面が出る。
許可後 `Send Notification` で通知が表示されれば成功。

<img src="/img/laravel-webpush/laravel-webpush.png">

## 既存プロジェクトでの対応方法
古いバージョンで作ってる場合はboxを作り直したほうが早いはず

1. DB などのバックアップを取る。次でデータは全部消える。
2. vagrant destroy
3. vagrant up
4. DB を戻す
