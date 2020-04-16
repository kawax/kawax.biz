---
title: "Laravel7以降、シンプルなAPIラッパーはHTTP Clientで十分かも"
date: 2020-04-12T14:54:33+09:00
categories: ["Laravel"]
draft: false
---

Laravel用に色々ラッパーライブラリ作ったけどなんのためかというとテストしやすさのため。
Guzzleそのまま使うとテストが面倒。
Laravel用に作ってFacade使うとテストしやすい。
でもLaravel7のHTTPクライアントはそれ自体がGuzzleラッパーでテストしやすいので別パッケージとして作らなくてもいい。
（複雑なことをするなら作っていいけど）

https://laravel.com/docs/7.x/http-client
https://readouble.com/laravel/7.x/ja/http-client.html

## 例としてQiita用を作る
https://qiita.com/api/v2/docs

https://github.com/kawax/qiita

## ServiceProvider
一番の肝はServiceProvider。AppServiceProviderでもいいしQiitaServiceProviderを作ってもいい。
Macroableで自由に拡張できるので`qiita()`メソッドを作ってトークンとbaseUrlを設定済のHttpクライアントを返す。

```php
use Illuminate\Support\Facades\Http;
use Illuminate\Support\ServiceProvider;

class QiitaServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Http::macro('qiita', function ($token) {
            return Http::withToken($token)->baseUrl('https://qiita.com/api/v2/');
        });
    }
}
```

PHP7.4なら`fn()=>`でもいいけどLaravel7はPHP7.2以上なのでこういう記事ではまだ7.2に合わせるべき。
各自のプロジェクトでは自由。

## 使い方
トークンは https://github.com/kawax/socialite-qiita を使って取得。

## ユーザーの記事一覧
`GET /api/v2/authenticated_user/items`

`Http::qiita()`でトークン設定して後はドキュメント見てメソッドとパスとパラメータを合わせる。

```php
    public function __invoke(Request $request)
    {
        $items = Http::qiita($request->user()->token)
                     ->get(
                         'authenticated_user/items',
                         [
                             'per_page' => 100,
                         ]
                     )->json();

        return view('home')->with(compact('items'));
    }
```

## 記事を削除
`DELETE /api/v2/items/:item_id`

```php
    public function __invoke(Request $request, string $id)
    {
        $res = Http::qiita($request->user()->token)
                   ->delete('items/'.$id);

        return redirect()->route('home');
    }
```

ちなみに非公開にするにはtitleとbodyが必須っぽいので一旦取得後titleとbodyとprivateを指定して更新すればいい。