---
title: "Laravel ルーティングファイルは増やしていい"
date: 2019-02-02T11:02:29+09:00
categories: ["Laravel"]
draft: false
---

Vue + Vue Router + Vuex + Laravelで写真共有アプリを作ろう (4) 認証API | Hypertext Candy
https://www.hypertextcandy.com/vue-laravel-tutorial-authentication

これの間違い。
`routes/api.php`をwebミドルウェアで使うためにRouteServiceProviderを変更してるけどこれはやめたほうがいい。
フレームワークはルールなので普通と違うことをするならそれなりに理由が必要。
例えば後から何も知らない人が参加して気付けるか？と考えると難しい。それどころか変更した本人でさえ数ヶ月後には忘れる。

ではどうすればいいのか。（web.phpに書けば終わるけど他の方法）
RouteServiceProvider::map()の`//` このたった2文字がヒントだけどルーティングファイルは好きに増やしてカスタマイズしろってこと。

```php
    /**
     * Define the routes for the application.
     *
     * @return void
     */
    public function map()
    {
        $this->mapWebRoutes();

        $this->mapApiRoutes();

        //
    }
```

`routes/`にファイルを作ってRouteServiceProviderで読み込む。
今回なら内部用のAPIをwebミドルウェアで使うので`routes/internal_api.php`とでもする。中身の書き方はapi.phpと同じ。

RouteServiceProviderは

```php
    /**
     * Define the routes for the application.
     *
     * @return void
     */
    public function map()
    {
        $this->mapWebRoutes();

        $this->mapApiRoutes();

        $this->mapInternalApiRoutes();

        //
    }

    protected function mapInternalApiRoutes()
    {
        Route::prefix('api')
             ->middleware('web')
             ->namespace($this->namespace)
             ->group(base_path('routes/internal_api.php'));
    }
```

何も知らない人が見た場合。
RouteServiceProviderの変更には気付かなくても問題ない。
`routes/`は絶対に見るので見慣れないファイルがあれば必ず気付く。
webミドルウェアなことなどは`routes/internal_api.php`のコメントにでも書いておけばいい。


将来的に外部用のAPIを作るようになっても`routes/api.php`を普通に使える。
webミドルウェアに変えてたら面倒なことになる。

## web.phpかapi.phpかは認証方法で分ける
たぶん一番大元の間違いは「json返すものがAPI」という思い込み。
APIだからapi.phpに書く→apiミドルウェアが適用されてなにもかも間違った方向に。

Laravelのweb/apiはroutes、app/Http/Kernelのミドルウェア、config/auth.phpのguardsと色々出てくるけど偶然同じなのではなく全部セットと考える。
routesを最初と考えると間違える。完全に逆で認証から考える。

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'token',
        'provider' => 'users',
    ],
],
```

一番普通なブラウザでログインしてるのはSessionGuard
→セッション使うのでwebミドルウェア
→web.php
認証の指定は`auth`。省略してるだけなので正確には`auth:web`

APIはTokenGuard（デフォルトなのにドキュメントには何も書かれてない）
→セッションは使わないのでapiミドルウェア
→api.php
認証の指定は`auth:api`
Passportなどを使う場合でも同じ。
