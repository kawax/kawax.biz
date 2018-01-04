---
title: "[WIP]Shopify"
date: 2018-01-02T19:26:54+09:00
categories: ["Shopify"]
draft: false
---

ネットショップ・サービス  
https://www.shopify.jp/

パートナー登録  
https://www.shopify.jp/partners

開発者向けの情報。APIは普通なので読めば大体分かった。  
https://developers.shopify.com/

Laravel 用の Shopily App パッケージも見つけた。APIパッケージもいくつかあるし十分普及してそう。  
https://github.com/ohmybrew/laravel-shopify

`netlify` とか `shopify` とかこのネーミングはどこから流行ってるんだろう…。

## Shopify App / Admin API
ショップ管理向けのAPI。別のショップと在庫を同期するような機能を作りたいならこっち。

## Storefront API
外部の任意のサイトにショップ機能を追加するみたいなAPI。

## Laravel で作る
`ohmybrew/laravel-shopify` の Wiki に色々書いてる。  
https://github.com/ohmybrew/laravel-shopify/wiki

Shopify 側のパートナーダッシュボード
- Development stores: 開発用のストアを作る
- Apps: アプリを作る。

Shopify App は別サーバーで動いてるウェブサイトでしかない。普通にOAuthで認証。Shopify のダッシュボードから表示してるだけ。

開発中はローカルサーバーでも問題ない。httpsは必須。App Storeに公開しなくても使える。多少警告は出るけど。`This app is not listed in the Shopify App Store. Contact the app developer for support.`

アプリを作ってAPIで商品情報取得まではできたので後はなんでもできる。

<img src="/img/shopify/shopify1.png">

## Private Apps
- https://help.shopify.com/manual/apps/private-apps

上で作ったアプリは公開すればどのストアでも使える。プライベートアプリは一つのストア専用。  
Shopify Appとして公開するための機能が不要でAPI Keyとパスワードを使っていきなりAPIを使える。  
自分のストアに対してAPIで何かしたいだけならこっちのほうが簡単。

非公開でアプリ使うことをプライベートアプリと言ってるのかと思ったけど違った。  
「アプリ管理」の下の方に小さく `Working with a developer on your shop? Manage private apps` と書いてあるだけなので気付かなかった。

## App Proxy
- https://github.com/ohmybrew/laravel-shopify/wiki/App-Proxies
- https://help.shopify.com/api/tutorials/application-proxies

ストアのドメイン下でアプリを表示できる。ShopifyのAPI関係なくてもブログでもなんでも表示できるわけだから活用方法は色々ある。
