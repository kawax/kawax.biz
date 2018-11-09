---
title: "Laravel5.7へアップグレード"
date: 2018-09-05T10:27:09+09:00
categories: ["Laravel"]
draft: false
---

5.5LTSではなく5.6使ってる人なら半年ごとにアップデート作業する覚悟はできてるだろうからさっさと進める。

## 準備
いつもの見る所。

- https://laravel.com/docs/5.7/releases
- https://laravel.com/docs/5.7/upgrade
- https://github.com/laravel/laravel/compare/5.6...master
- https://github.com/laravel/laravel から最新版をzipでダウンロード。

## Bladeのor
とはいえ5.7は大きな変更はほとんどないのでフレームワーク側はcomposer.jsonを5.7にするだけでほぼ終わる。   
確実に壊れるのはBladeの`or`だけ。3月頃には分かってた変更なので対応する時間は十分にあっただろう。  
今からならviews以下を検索して書き換えればいいだけ。ifのorと間違えないようにだけ注意。

```php
// Laravel 5.6...
{{ $foo or 'default' }}

// Laravel 5.7...
{{ $foo ?? 'default' }}
```

## プロジェクト側
GitHubの比較ツール見ながら書き換え。

## QUEUE_CONNECTION
QUEUE_DRIVER→QUEUE_CONNECTION。アップデートなら変えなくてもいいと思う。

## VerificationController
使わないなら不要。

## Middleware\Authenticate
'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
↓
'auth' => \App\Http\Middleware\Authenticate::class,

app以下に変更されてるのでコピーしてくる。

## public/svg
404ページ用とかの画像。  
エラーページ用のviewを変更してる場合はここを参考に。  
https://github.com/laravel/framework/tree/5.7/src/Illuminate/Foundation/Exceptions/views

## resources/assets
assetsがなくなってresources直下にjsとsass。  
webpack.mix.jsも`assets`削除。

## 終わり
プロジェクト側も細々と変わってるように見えて実際の動作にはほとんど影響のない変更ばかり。  
全部合わせる必要はないので関係ない箇所は無視すればいい。

## その後
ほとんどのプロジェクトで5.7化完了。関係ない箇所を無視すれば一つ10分以内に終わる。  
アップデート作業の手間をかけたくないのは5.5のまま残してる。  
サードパーティパッケージで`5.6.*`で指定しててcomposer updateできないのが一番困る。

## さらにその後
後からアップグレードガイドに追記されると困る。  
ファイルキャッシュ用に`storage/framework/cache/data/`作るように追記されたけどこの変更はそもそも5.7じゃないような…。  
configはここで変更されてるので5.4。  
https://github.com/laravel/laravel/commit/402b12f9159498e9be25e2a77939281fb35f6e13
