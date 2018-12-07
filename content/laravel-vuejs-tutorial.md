---
title: "Laravel + Vue.js 普通の入門"
date: 2018-12-07T16:39:26+09:00
draft: false
---


## 前提
- SPA(Single Page Application) **ではない**
- SSR(Server Side Rendering) **ではない**
- Laravelの基礎は分かってる人向け。
- Vue.js初心者はガイドの「基本的な使い方」「コンポーネントの詳細」までは必ず先に読んでおく。最初のほうのCDNから使う方法はLaravelでは一切使わないので一応知識として入れておく程度でいい。https://jp.vuejs.org/

## 環境
- Laravel 5.7
- Vue.js 2.5.x
- ローカルサーバー php artisan serve
- node.js 11.1 / npm 6.4
- Laravel Mix 3.0

バージョンが変われば当然使い方も変わるので丸暗記ではなく考え方を覚える。

## 作るもの
入門用なら簡単な例でもいいけど少しだけ実践的にLaravel Notificationのデータベース通知を表示するメニューバー部分を作る。あくまで想定なのでLaravel側は詳しくは作らない。

<img src="/img/laravel-vuejs-tutorial/laravel-vuejs-tutorial1.png">


どこをVueで作るか？と考える場合はまずはログインしてるユーザーにしか見えない部分をVueで作る。SSRしないとGoogleクローラーから見えないことが問題になるけどログイン後のユーザー向けなら最初から見えないので関係ない。

ページ全体をVueで作るとか考えなくていい。部品の一つをVueで作る所から始める。

## プロジェクト作成

```
laravel new laravel-vuejs-tutorial && cd $_
php artisan make:auth
npm i
```

普通にVueを使う場合は`make:auth`まで必要。Vueには‎`resources⁩/views⁩/layouts⁩/app.blade.php`が大事。

`resources⁩/js/app.js`の`#app`は

```
const app = new Vue({
    el: '#app'
});
```

`layouts⁩/app.blade.php`の`id=app`のことを指している。

```
<div id="app">
</div>
```

他は全部用意されてるので気にしなくていい。
ざっと見る所

- package.json
- webpack.mix.js
- resources⁩/js/

`php artisan serve`でローカルサーバーを起動して`http://127.0.0.1:8000/`で表示されるのを確認。

ログインなしでもHomeを表示できるようにHomeControllerの`$this->middleware('auth')`をコメント。
この記事用の処置なので実際には不要。(Homesteadで開発する場合はユーザー登録してログイン状態で表示。)

`app/Http/Controllers/HomeController.php`

```php
class HomeController extends Controller
{
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        // $this->middleware('auth');
    }
}
```

`http://127.0.0.1:8000/home`で表示できればここまでは完了。
`php artisan serve`は開発中起動したまま。

## Laravel側APIの用意
`routes/web.php`から簡易的にjson返すだけ。

```php
Route::get('/api/notifications', function () {
    return [
        [
            'id'      => 1,
            'message' => 'message 1',
        ],
        [
            'id'      => 2,
            'message' => 'message 2',
        ],
    ];
});
```

ブラウザで`http://127.0.0.1:8000/api/notifications`を表示すればjsonが見える。

### routes/api.phpでない理由
api.phpに書くなら本来は認証も含めてこう書く。

```php
Route::middleware('auth:api')->get('/notifications',
```

api用の認証方法を気にする必要がある。api_token渡したりPassport使ったり。今回のように同じサイトでブラウザから使用するだけであればweb.phpに書いてweb用の認証=セッションに任せれば何も気にしなくていいはず。

```php
Route::middleware('auth')->get('/api/notifications',
```

## Vue.js側でVueコンポーネント
‎`resources⁩/js/components/`に`NotificationsComponent.vue`を作る。APIで取得して表示してるだけの簡単なもの。
今後もまず最初にVueコンポーネントを作る。

```
<template>
    <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-toggle="dropdown"
           aria-haspopup="true" aria-expanded="false">
            通知 <span class="badge badge-pill badge-primary">{{ notifications.length }}</span>
        </a>
        <div class="dropdown-menu" aria-labelledby="navbarDropdown">
            <a class="dropdown-item" v-for="notification in notifications">{{ notification.message }}</a>
        </div>
    </li>
</template>

<script>
    export default {
        data () {
            return {
                notifications: [],
            }
        },
        mounted () {
            this.getNotifications()
        },
        methods: {
            async getNotifications () {
                const res = await axios.get('/api/notifications')
                if (res.status === 200) {
                    this.notifications = res.data
                }
            },
        },
    }
</script>
```

app.jsで登録。example-componentと同じ場所。

```
Vue.component('example-component', require('./components/ExampleComponent.vue'));
Vue.component('notifications-component', require('./components/NotificationsComponent.vue'));
```

`npm run dev`でビルド。public内に出力される。

Vueコンポーネントを作る→app.jsで登録→ビルドの流れ。Laravel+Vue.jsで覚えるべきはこの流れのみ。


### 慣れたら
`npm run watch`を起動したままにしていればVueファイルを更新するたびに自動でビルドされるので本格的な開発中はwatchを使う。

### 更に慣れたら
Vueファイルはそんなに触らないので最初から`npm run prod`しか使わず本番用のファイルしかビルドしない。

## Laravel側のBladeを更新
`resources⁩/views⁩/layouts⁩/app.blade.php`

```
<!-- Left Side Of Navbar -->
<ul class="navbar-nav mr-auto">
    <notifications-component></notifications-component>
</ul>
```

ログインしてないので左に置いてるけど本来はログイン中のみ表示されるように右側。もしくは@auth内。
これで通知が表示されるので完成。

`resources⁩/views⁩/home.blade.php`にexample-componentを書けばこれも表示される。

```
<example-component></example-component>
```

Vueコンポーネントを作れば後はhtmlタグの感覚で使えることを覚える。
どこでもではなく`id=app`内のみ。

```
<div id="app">
</div>
```

`layouts⁩/app.blade.php`が↑こうなってるので`app.blade.php`を継承したviewではどこでも使える。

```
@extends('layouts.app')

@section('content')
(この中ならどこでも使える)
@endsection
```

新しいviewを作るときは必ず`@extends('layouts.app')`で継承。
このために最初の`make:auth`が大事。

### 慣れたら
`resources⁩/views⁩/welcome.blade.php`には`id=app`がないのでVueコンポーネントは使えない。
でも`<script src="{{ asset('js/app.js') }}" defer></script>`と`<div id="app"></div>`を追加すれば使える。

これと同じように自分でVue使うための準備ができるなら`make:auth`は不要。

## 本番公開用の準備

webpack.mix.js。キャッシュ対策のバージョン付け。

```
mix.js('resources/js/app.js', 'public/js')
   .sass('resources/sass/app.scss', 'public/css')
   .version();
```

`npm run prod`で本番用ビルド。

`layouts⁩/app.blade.php`の`asset()`を`mix()`に変更。jsとcss2つ。

`version()`を付けると`public/mix-manifest.json`の出力が`?id=`付きに変わる。viewに反映させるためにmix()ヘルパーに変更。

```
{
    "/js/app.js": "/js/app.js?id=xxx",
    "/css/app.css": "/css/app.css?id=xxx"
}
```

今後もjs/cssを変更した時は`npm run prod`で再ビルドを忘れないようにする。
デプロイ時に毎回再ビルドしてもいいけどLaravelプロジェクトでそこまでやるのは無駄だと思うので自分ではやらない。
ここは各自の好みで決めること。

### 慣れたら
最初のプロジェクト作成時にここまで変更。

## 新しいページを作る時
今回はメニューだから全体で使えたけど特定のページ用のVueコンポーネントを作る場合。
普通にLaravel側でルーティング・コントローラー・viewを作る。
Vueコンポーネントを作る→登録→ビルドしてviewで使う。
この流れさえ分かっていれば後は簡単。

Vueコンポーネント内のことはもうLaravelとは関係なくVue.jsの使い方レベルの話。

## よく見る間違い

### Vueコンポーネントを使わない
`make:auth`しないまま使い始めるとVueのドキュメント見て
CDNで読み込み、script内でVueのコードを書き、Bladeとの`{{ }}`の重複を避けるために`@{{ }}`を使う。
これ全部間違い。

htmlとVue.jsとBlade(PHP)をごちゃまぜに書いたら後からの変更がしにくい。htmlとPHPを混ぜて書いて失敗した過去を知っていればさらにVue.jsを足したらどんな地獄になるかは誰でも分かる。
そうならないためにLaravelでは最初からVueコンポーネント使うように用意されている。

Laravelのドキュメントには`@{{ }}`のことが書いてあるけどこれはVueのために作った機能ではない。Vue誕生前から存在している。Vueでは今はVueコンポーネント使えば不要なので関係ない。
https://readouble.com/laravel/5.7/ja/blade.html#blade-and-javascript-frameworks

## 発展的内容
入門時点では気にしなくていい。

### ブロードキャスト
通知を例にしたのはLaravelのブロードキャストを使えばリアルタイムで通知を受け取れるようになるから。SPAまでやらなくてもVueコンポーネント一つでそのくらいはできる。
https://readouble.com/laravel/5.7/ja/broadcasting.html

### SSR
SSRするならNuxt.js使ったほうが早い。Vue.jsとはいえLaravelでの普通の使い方とはかなり変えることになる。
https://github.com/kawax/nuxt-socialite-demo

## 終わり
https://github.com/kawax/laravel-vuejs-tutorial
