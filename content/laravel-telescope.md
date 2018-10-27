---
title: "Laravel Telescopeを試す"
date: 2018-10-24T11:29:55+09:00
categories: ["Laravel"]
draft: false
---

ローカルでの開発時に役立つ情報を色々表示するツール  
https://github.com/laravel/telescope

## 環境
- Laravel 5.7
- Telescope 0.1.3

## インストール
新規のLaravelプロジェクトにインストールする前提。既存プロジェクトでも同じはず。  
https://github.com/kawax/telescope-demo

```
composer require laravel/telescope --dev
```

```
php artisan telescope:install
```

これで追加されるのはこんな感じ。  
https://github.com/kawax/telescope-demo/commit/294e601b522901b1d82ce9fe5aab54b9eda2eb7c

```
php artisan migrate
```

データベースが必要なので`php artisan serve`では動かしにくい。Homesteadでのみ動作確認。ローカルのみなので本番環境のデータベースへの影響はない。

後は`http://local/telescope`で表示するだけ。

`/telescope`を変更したい場合などの設定は`config/telescope.php`

## ダッシュボード認証
ローカルで使うものなので基本的にはこの設定は不要。

## Dumps
新規プロジェクトだと簡単なことしか試せないけど気付いたことを残す。  
`dd()`は使えない。`dump()`のみ。何も表示されずTelescopeにのみ表示される。Laravel5.7で追加されたdump-serverと同じ仕様。

## Mail
他の機能も使ってる既存プロジェクトに追加して試したけどしっかり記録される。特にメールは今までMailHogなどを使ってたけどTelescopeに集約されるのでいいかもしれない。

## 本番環境
ローカルのみなので本番環境ではインストールされないけどTelescopeServiceProviderが残ってるのでエラーになる。  

```
use Laravel\Telescope\TelescopeApplicationServiceProvider;

class TelescopeServiceProvider extends TelescopeApplicationServiceProvider
```
インストールしてないからTelescopeApplicationServiceProviderがないのに継承してる。

`config/app.php`のTelescopeServiceProviderをコメントにしておく。
```
//App\Providers\TelescopeServiceProvider::class,
```
この辺りはまだ出たばかりで問題が把握されてないんだろうから上手く解決されるのを待つ。

### とりあえずの解決方法
- TelescopeServiceProvider.phpは残す
- config/app.phpはコメント

AppServiceProviderのboot()に追加。
```php
if ($this->app->environment('local')) {
    $this->app->register(TelescopeServiceProvider::class);
}
```

https://github.com/laravel/telescope/issues/154#issuecomment-432938123

これでローカルでのみ有効。

## アップデート時
assetsが更新されてる場合はこれも。

```
php artisan vendor:publish --tag=telescope-assets --force
```
