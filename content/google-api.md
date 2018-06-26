---
title: "Google APIシリーズ"
date: 2018-06-23T15:48:36+09:00
categories: ["Laravel"]
draft: false
---

PHPからGoogle APIを使うにはこのライブラリを使えばいいんだけど  
https://github.com/google/google-api-php-client  
他の言語と同時に自動生成してるせいかそのままだとものすごく使いにくい。  
https://github.com/google/google-api-php-client-services  
Golang用とかそれぞれ1ファイルにまとめてるので巨大。  
https://github.com/google/google-api-go-client

使いにくいなら自分で使いやすいラッパーを作ればいいということでLaravel用にはいくつか作ってる。

## Google Sheets
https://github.com/kawax/laravel-google-sheets  
最初なのでそんなに使いやすくなってない。どうもAPIv4では実現したかった機能を作るのが難しいので諦めた。
すでに存在するスプレッドシートを読み込むのが主な用途。

## Google Photos
https://github.com/kawax/laravel-google-photos

## AdSense
プロジェクトのみ  
https://github.com/kawax/laravel-google-adsense-project

## 未公開
GmailやSearchConsoleも使ってるけど公開してない。そのまま使っててパッケージに分離できるような状態ではないので作り直さないと無理。

## SearchConsole
プロジェクトだけ公開用に作り直した。  
https://github.com/kawax/laravel-google-searchconsole-project

## ついでにGoogle APIの基本的な使い方
まずDeveloper Consoleでプロジェクトを作ってAPIを有効化する。  
https://console.developers.google.com/  
そもそもここが一番分かりにくい。

### 認証情報
APIキー、OAuth、サービスアカウントと3種あるけど基本的にはOAuthを選べばいい。  
サービスアカウントとかいかにも自分のアカウントのデータを自動で読むのに使えそうだけど実は違う。  
プロジェクトを作ったアカウントとサービスアカウントのアカウントは別のアカウント扱い。  
Sheetsなら共有してサービスアカウントからでも使えるけどAdSenseはサービスアカウント非対応なので使えない。  
分かりにくいのでOAuthでいい。

### Google_Client
google-api-php-clientのまま使うなら。  
credentials.json使ってるけどclient_id,client_secretを指定でもいい。  
https://github.com/google/google-api-php-client#authentication-with-oauth  
Laravelなら  
https://github.com/pulkitjalan/google-apiclient  

認証情報をセットしたGoogle_Clientを作る所まで理解できていれば新しいAPIが登場してもすぐ使える。

### token
自分のアカウントのデータだけ読むとしてもOAuthで認証してaccess_tokenとrefresh_tokenを取得して保存して毎回access_tokenをrefreshすればいい。  
普通にやるとrefresh_tokenが空。  
Google_Clientを作る時にこう設定しないとrefresh_tokenが取得できない。  
ここが一番はまりやすい。  

```php
'access_type'      => 'offline',
'approval_prompt'  => 'force',
'prompt'           => 'consent',
```

### 取得後のデータの扱い
APIから取得するデータはGoogle_なんとかで専用のクラスになってることが多いけど扱いにくい。    
ほとんどはGoogle_Modelを継承してるのでさっさと`toSimpleObject()`して普通のオブジェクトにしたほうが扱いやすい。  
https://github.com/google/google-api-php-client/blob/master/src/Google/Model.php
