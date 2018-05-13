---
title: "Google Photos API for Laravel"
date: 2018-05-13T09:14:46+09:00
categories: ["Laravel"]
draft: false
---

Google PhotosのAPIが公開されていたのでLaravel用のパッケージを作る。  
https://developers.google.com/photos/

GoogleのAPIはこれだけで使えると言えば使えるけど結構面倒なのでLaravelらしくシンプルに使えるようにするのが目的。  
https://github.com/google/google-api-php-client

Google_Clientのラッパーはすでにあるのでこれを使う。  
https://github.com/pulkitjalan/google-apiclient

OAuth認証にはSocialite。Socialite使わなくてもいいけどLaravelなら使ったほうが楽だろう。

## 事前準備
https://developers.google.com/console

OAuth用のClient ID、Client Secretを取得。
いくらでも情報あるので省略するけどGoogle API使う時はこの事前準備してる段階が一番面倒。  

APIは`Photos Library API` `People API` `Google+ API`を有効化。
PeopleとGoogle+はSocialite用。どちらかは不要かもしれないけど無効化して動かなければ有効化。

## 開発中
普通にLaravelプロジェクト作って`lib/laravel-google-photos/`にパッケージ用ファイルを置いて
composer.jsonのautoloadで設定。

```
"Revolution\\Google\\Photos\\": "lib/laravel-google-photos/src/"
```

最終的には消すけどこうしておけば後でパッケージに分離してもそのまま使える。

## Facade
Socialiteでログインしてaccess_tokenを取得できる所までできたら本格的に作っていく。
今回はシンプルなラッパーなので難しいことはしてない。

Laravel用ならFacadeからメソッド数個で使えるようにするのが最終目標。

```php
$albums = Photos::listAlbums();
```

実際にはGoogle API使う場合は

- Google_Clientにtokenセット。
- そのGoogle_ClientからGoogle_Serviceを作り直す。
- Google_Serviceを使う。

という流れなので
Google Facade使ってもこのくらいになりがち。

```php
$token = $request->user()->access_token;

Google::setAccessToken($token);

$photos = Google::make('PhotosLibrary');

$albums = Photos::setService($photos)->listAlbums();
```

今回は完全にLaravel用と割り切ってPhotosにsetAccessToken()用意したので最終的にはこのくらいで。

```php
$token = $request->user()->access_token;

$albums = Photos::setAccessToken($token)->listAlbums();
```

テスト時のモックがこれだけで済むのでこれならまぁいいかと。

```php
Photos::shouldReceive('setAccessToken')->once()->andReturn(m::self());
Photos::shouldReceive('listAlbums')->once()->andReturn((object)['albums' => []]);
```

一応Laravel以外でも使おうと思えば使えるはずだけどGoogleの公式ライブラリだけ使ったほうがいい。

## Trait
Facadeでも普通にLaravelらしくていいんだけどそろそろ飽きてきたのでもうちょっと何かないかと考えて閃いた。

まずUserモデルにPhotosLibrary Trait追加。

```php
namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

use Revolution\Google\Photos\Traits\PhotosLibrary;

class User extends Authenticatable
{
    use Notifiable;
    use PhotosLibrary;

    protected $dates = [
        'created_at',
        'updated_at',
    ];

    /**
     * Get the Access Token
     *
     * @return string|array
     */
    protected function photosAccessToken()
    {
        return [
            'access_token'  => $this->access_token,
            'refresh_token' => $this->refresh_token,
            'expires_in'    => $this->expires_in,
            'created'       => $this->updated_at->getTimestamp(),
        ];
    }
}
```

`photosAccessToken()`でtokenを返す。abstractなので必須。

Userモデルから`photos()`が使えるようになる。

```php
$albums = $request->user()
                  ->photos()
                  ->listAlbums();
```

Notificationと大体同じ。
「Traitで有効化してUserモデルから使う」をできるようにしたら非常にLaravelらしくなった。

やってることは同じなのでテスト時のモックも同じ。

## アップロード
Googleのライブラリにアップロード部分はないので自分で行う。
指定のエンドポイントにPOSTするだけなので簡単。ライブラリ側で用意するまでもない。
今回初めて気付いたけどGoogle_Clientの`authorize()`で認証済み`GuzzleHttp\Client`が返ってくる。
headersにtokenも自動で設定してくれるのでこれを使えばいい。

アップロード処理は実際にファイルをアップロード後返ってくる`uploadToken`を使ってフォトライブラリに追加する2ステップが必要。

## Macroable
あのメソッドが足りない？Macroable対応してるので勝手に生やすんだ。

## 終わり
完成品

- https://github.com/kawax/laravel-google-photos
- https://github.com/kawax/laravel-google-photos-project
