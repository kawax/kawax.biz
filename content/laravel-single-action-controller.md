---
title: "1コントローラー1アクション"
date: 2017-12-11T11:36:44+09:00
categories: ["Laravel"]
draft: false
---

## 最初に結論
リソースコントローラー以外は（なるべく）1コントローラー1アクションで作る。

以前にRailsの偉い人が言ってたような話。  
[DHHはどのようにRailsのコントローラを書くのか | プログラミング | POSTD](http://postd.cc/how-dhh-organizes-his-rails-controllers/)

## Laravel でのやり方

シングルアクションコントローラーとして作る。  
https://readouble.com/laravel/5.5/ja/controllers.html#single-action-controllers

### コントローラー
`__invoke()` で実装することによりこのコントローラーではこのメソッドのみが実行される。実際には他のメソッドを追加してもいいけど追加したくなってる時点でどこかおかしいので考え直す。

ドキュメントのままのサンプル。

```php
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class ShowProfile extends Controller
{
    /**
     * 指定ユーザーのプロフィール表示
     *
     * @param  int  $id
     * @return Response
     */
    public function __invoke($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

なお、 `__construct` は使ってもいい。

### ルーティング
```php
Route::get('user/{id}', 'ShowProfile');
```

通常なら `'ShowProfile@show'` などと書く所を省略。

Railsの記事と同じように `index` などのメソッドのみでもいいけど `__invoke` ならこのコントローラーではこれ以外のことはしないと分かりやすいので。

## 感想と余談
しばらくこれで作ってみたけどやはり分かりやすさは大事。コントローラーを小さくするのは基本中の基本。

Laravel は規約が少ないとは言われるけど決めないといけないのはモデルの置き場所くらいで他は Laravel のデフォルトに従えば十分。

もちろんプロジェクトの規模によるので最終的には自分で決めるのが大前提。

「Laravel ではモデルは `app/` 直下に置くのが流儀」なんて書かれてるのを見たことあるけどもちろんそんなわけない。
User.phpとかPost.phpとか増えていくのを見たら普通に考えればおかしいと分かる。
`app/Model/` でも `app/Eloquent/` でもなんでもいいので自分の好きな名前でディレクトリを作れ、が Laravel。
https://readouble.com/laravel/5.5/ja/structure.html
作る時は `php artisan make:model Model/Post`
使う時は `use App\Model\Post;`
PSR-4とcomposerに従えばアプリ内のどこからでも使える。

どうも複雑にしてる人は可能性の低い将来の心配をしすぎている。他のフレームワークに変えるかもしれないので Laravel の便利な機能を使わない、DBエンジンを変えるかもしれないから Eloquent を使わない…などなど。現実的にはそんな状態になったらほとんど作り直しだから役に立つことは少ないと思ってる。PSR や composer のない時代ならともかく今はなぁ…。

Advent Calendar なのでついでに色々書こうとしてたけどこれ以上はやめておく。

## 補足
Advent Calendar 用に書いてたけど Kobito 終了により Qiita には書きにくくなりそうなのでここにも残しておく。  
[Laravel Advent Calendar 2017 - Qiita](https://qiita.com/advent-calendar/2017/laravel)
