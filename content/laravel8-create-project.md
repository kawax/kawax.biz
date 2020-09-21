---
title: "Laravel8時点の新規プロジェクト作成方法"
date: 2020-09-21T12:57:11+09:00
categories: ["Laravel"]
draft: false
---

## laravel/installerをv4.0以上に更新
```
composer global require laravel/installer
```
重要なのでしばらくは何度でも書いたほうが良さそう。
**旧バージョンのインストーラーは使えなくなるので今すぐ更新が必要。**

```
laravel -V
Laravel Installer 4.0.4
```

この記事は4.0.4時の情報。

## Jetstream付きで作成
```
laravel new laravel8-jet --jet
```
Livewire or Inertiaやチーム機能を使うか聞かれる。

Laravel8以降の新規でユーザー登録や認証機能を使うなら基本的にはJetstreamを使う。

LivewireかInertiaかの選択が重要だけど分からないならLivewireでいいはず。
Livewireで作ってLivewireの機能を無視すれば今までとほとんど変わらない。

`--stack`で最初からLivewireを指定できる。結局チーム機能を使うかは聞かれる。
```
laravel new laravel8-livewire --jet --stack=livewire
```
`--teams`まで付ければ何も聞かれない。
```
laravel new laravel8-livewire --jet --stack=livewire --teams
```

## Jetstreamなしで作成
```
laravel new laravel8
```

ユーザー登録やログインは自分で作るとか、
認証はSocialiteしか使わない、Jetstreamの他の機能も使わない場合はなしでもいい。
API用認証が必要ならSanctumを単独でインストールすればいい。

## laravel/uiは非推奨
一応Laravel8でもまだ使えるけど公式に「使うべきではない」と書かれてるので使わないほうがいい。
```
laravel new laravel8
composer require laravel/ui
php artisan ui vue --auth
```

あくまでも旧バージョンからアップグレードしたプロジェクトで使う用。

## インストーラーを使わずcomposer create-projectで作成
4.0からインストーラーもcreate-projectを使ってるだけ。Jetstreamなしと同じ。
```
composer create-project --prefer-dist laravel/laravel laravel8 "8.*"
```
Jetstreamを使うなら手動でインストール。
```
composer require laravel/jetstream
php artisan jetstream:install livewire
```
laravel8がプロジェクト名。最後の"8.*"がバージョン。
つまりLaravel6を指定して作成することもできる。
```
composer create-project --prefer-dist laravel/laravel laravel6 "6.*"
```
Laravel8が変わりすぎて書籍などの情報が使えなくなって困ってる人はLaravel6で作ってもいい。

## もう関係ない昔の話
Laravel5.8まではmake:auth
```
laravel new laravel5
php artisan make:auth
```
Laravel6ではlaravel/uiが分離
```
laravel new laravel6
composer require laravel/ui
php artisan ui vue --auth
```
Laravel7頃？インストーラーに`--auth`追加
```
laravel new laravel7 --auth
```
make:authでもuiでもプロジェクト作成後に一手間必要だったけど
`--auth`の時期だけは一発で全部用意されたプロジェクトが作成できた。
説明も簡単。
せっかく簡単になってたのに`--auth`が削除されて`--jet`に完全に置き換えられた。
今後は`--jet`で説明するだけだけど初心者にJetstreamは難しい気がする。