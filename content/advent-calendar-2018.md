---
title: "Advent Calendar 2018 / Laravelの小ネタ25連"
date: 2018-11-13T11:27:54+09:00
categories: ["Laravel"]
draft: true
---

誰か抜けたのか1の枠が空いてたので久しぶりにQiitaに書く。2には最近Qiitaに書いてないと書いてるけど。

## 1. 新規プロジェクトを作るときはphp artisan make:authまでやる
https://readouble.com/laravel/5.7/ja/authentication.html  
app.jsの`#app`が何を指しているかというと  
```
const app = new Vue({
    el: '#app'
});
```
make:authで作られる`layouts/app.blade.php`の`<div id="app">`  
https://github.com/laravel/framework/tree/5.7/src/Illuminate/Auth/Console/stubs/make

app.jsで指定してるのにwelcome.blade.phpには`<div id="app">`もcsrfもないので初期状態ではVue.jsが使える設定になってない。make:authまでやれば最初からVue.jsの使える環境ができあがる。

これが分かってなくてLaravelでのVue.jsの使い方を間違えてる例をよく見る。

もちろん理解してる人ならmake:authせずに自分で好きなように準備すればいいけど。  
分かってない内はLaravelの普通の使い方に従う。  

## 2. 普通
基本的にLaravelはユーザーが楽できるように色々整備されてるので普通に使えば効率よく開発できる。  
普通(=ベストプラクティス)を知った上で自己流にアレンジするならいいけど知らないまま無駄な作業してる人がものすごく多い。

## 3. Homesteadはプロジェクトごとにインストール
https://readouble.com/laravel/5.7/ja/homestead.html#per-project-installation  
Homesteadも導入に苦労してる例が多いけどプロジェクトごとにインストールしたほうが混乱しない。

## 4. HomesteadはBonjourを使うとhostsの設定なしで使える
https://kawax.biz/bonjour-vagrant/

vagrant ssh後にこれだけ。
```
sudo apt install avahi-utils
```

hostsが何十個にも増えてきたので最近はこれでやってるけど楽。

## 5. action([HomeController::class, 'index'])は使いにくい
https://readouble.com/laravel/5.7/ja/urls.html#urls-for-controller-actions

ルーティングでも同じ。useして使うか名前空間からフルで書く必要があるのでどういう場面で使う想定なのか分からない。
```
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

## 6. アップグレードガイドに後から追記される
5.7.0時から少し増えてる。さすがにアップグレードガイドに追記はやめてほしいけど5.7は大きな変更はなかったし追記分も影響はないのでまぁいい。
https://readouble.com/laravel/5.7/ja/upgrade.html

docsまでいちいちチェックするのは厳しい。  
https://github.com/laravel/docs

## 7. laravel/laravelはチェックすべき
たまに手動での対応が必要な変更が入る。  
https://github.com/laravel/laravel

## 8. .editorconfig
laravel/laravelのほうに追加されたのは今年。PHPはPSR-2しかないし.editorconfigに対応したエディタを使っていればコーディングスタイルを気にすることはほぼない。  
https://github.com/laravel/laravel/blob/master/.editorconfig

## 9. ログ収集にはPapertrailが便利
https://papertrailapp.com/  
Forgeでは前から対応してたけどLaravelでもLogの設定に増えた。  
https://github.com/laravel/laravel/commit/ea3afd1013c24b696eefdcd2097cc2eddeb82849#diff-0c5eb58e57861e3bdcc3a2f6ae493626

## 10. env()はconfigでしか使えない
https://readouble.com/laravel/5.7/ja/helpers.html#method-env

config外でenv()使ってる例をよく見るけど`config:cache`でキャッシュするとenv()はnullになる。  
面倒でも全部一旦configを経由する。  
config/my.php
```
return [
  'foo' => env('FOO'),
];
```

```
$foo = config('my.foo');
```

## 11. CarbonにはLaravel用のコードが含まれる
https://github.com/briannesbitt/Carbon
package:discover時に`Discovered Package: nesbot/carbon`が出てくるようになって気付いた。元々Laravel用というわけでもないのにわざわざ入れてくれてるんだな。  

## 12. Laravel mix v2.1はwebpack3
多少warningが出るようになってるけどいつwebpack4に対応するんだろう。軽く使う範囲なら現状でも問題ないけど早くwebpack4使いたいような人はmix捨てて直接webpack使うのが良さそう。

4にできない理由はあるようなので5までこのままかも。  
https://github.com/JeffreyWay/laravel-mix/pull/1495

## 13. js/cssは全部npmとmixで管理
最近見たLaravel製のサイトの多くが`public`下に直接css等を置いて使ってた。
```
<link href="/css/bootstrap.min.css" rel="stylesheet">
```
フロントのスキルが足りてない。昔のやり方をLaravelでもそのままやってる。

jQueryプラグインだろうと全部npmで管理。npmで管理できないほど古いjQueryプラグインなら捨てて他を探したほがいい。

どうしてもnpmで管理できない場合でもファイルはまず`resources`に置く。mixでのビルドで`public`にコピーしたり`mix.scripts()`で結合したりする。  
基本的に`public`以下は自分で触らずjs/css(+font)は全部mixを通してビルドしたファイルの出力先とする。  
画像なら直接置いてもいいけど自動で最適化とかするなら画像も`resources`。

## 14. Laravel Frontend Presetsはまだドキュメント入りしない
https://github.com/laravel-frontend-presets
`assets`が消えたのに対応できてないのもあるので今使うとエラーになるかも。どうせ1回しか使わないのでvendor以下を直接書き換えて実行してcomposer.jsonから消していい。

## 15. Dockerは本番用
開発環境のためだけに使っても意味はない。

デモアプリをコマンドコピペだけで起動できるようにするような用途もあるけどLaravelの場合は事前準備が必要なので向いてない。

## 16. Push to Deploy
Laravel使ってて未だにFTP使ってる人がいて本当に信じられないんだけどもはやgit pushからのデプロイが当たり前。  
5年以上前でもHerokuみたいなPaaSにgit pushでデプロイしてたことは手元のPC内に残ってるファイルからも確認できる。昔のHerokuはPHPに対応してなかったので別の所。いくつか転々としたけどほとんどサービス終了したので結局最後はAWS。

Railsチュートリアルの丸パクリでいいからHerokuにデプロイまでやるLaravelチュートリアルを誰か書いたほうがいい。

## 17. Laravel Telescopeを使ってて本番環境にデプロイするには一工夫必要
https://kawax.biz/laravel-telescope/

これが答え。  
https://github.com/laravel/telescope/issues/154#issuecomment-432938123

本番ではTelescopeは無効だけどREADME通りに使ってるとデプロイ時になってあれ？と気付く。

## 18. Novaを試した人はあまり見かけない
管理画面パッケージだけど有料だとな。  
https://nova.laravel.com/

一部でVoyager使ってるけどmigrationsが必要だしUserも`TCG\Voyager\Models\User`から継承でかなり依存させられる作り。いつまでも5.4以上対応としてるせいで5.7対応が遅かったので新規ではもう使わなそう。  
https://github.com/the-control-group/voyager

## 19. キューが使えて一人前
色々見てるとLaravelを分かってるかどうかはキューを使えるかが大きな分かれ目。  
当然の前提としてキューが使えるサーバーを用意するスキルが必要。  
Laravelにもキューの先にキューを前提とした機能がある。

## 20. 入り口は2つ
Laravelに慣れてきたら読むページ  
https://readouble.com/laravel/5.7/ja/lifecycle.html

`app/Http/Kernel.php` `app/Console/Kernel.php` カーネルなんていう名前だけで重要だと分かるファイルが2つあるのはLaravelの入り口が2つあるから。ブラウザからの表示にしか興味がないとルーティングからコントローラーを中心としたMVCだけに意識が向くけどartisanコマンドのConsoleも同列の扱いだと分かればLaravelへの理解が深まる。  
入り口によって`app/Http/`か`app/Console/`に分岐。HTTPリクエスト時にしか使わないファイルは`app/Http/`に置く。それ以外の`app/`下のファイルは両方で共通して使えるファイル。

## 21. MVCは忘れてもいい
GUIのMVCとWebMVCは完全に別物。MVCが双方向にやり取りするみたいなイメージ持ってる人が多いけどそんなことはできないので忘れていい。  
ルーティング→コントローラー→ビューの一方通行。  
モデルはそれ以外のほとんどで広すぎるので（自分の場合は）Eloquentをモデルとして扱う。  
サービスとかドメインとかLaravel用語ではないものはなるべく使わない。

Fluxとか別名付けたのは賢い。Fluxの流れをイメージしたほうが近い。

## 22. サービスコンテナ
https://readouble.com/laravel/5.7/ja/container.html  
Laravelの一番重要な部分だけど普通に使う範囲では別に知らなくてもいい。

Laravel用のパッケージ作るくらいになってやっと完全に理解したけど小ネタ記事で書けることではないのでまたいつか。

## 23. サービスプロバイダのdefer
https://readouble.com/laravel/5.7/ja/providers.html
`register()`で登録してるだけのServiceProviderなら`$defer = true`にできる。`boot()`で何かしてるならできない。  
ドキュメント通りでしかないけどこういうのも実際に使ってみて失敗しないと中々覚えない。

## 24. 5.4以前はすでにサポート終了
https://readouble.com/laravel/5.7/ja/releases.html
PHPも7.0まで今年末で終了なのでもう使ってはいけない。

## 25. LTS
次のLTSが5.9で2019年夏頃だとするともう来年なので今からなら5.7使えばいいと思う。  
最近は大きな変更も少ないので5.8→5.9と上げればいい。5.1ほどではないけど5.5からは面倒。  
今の状況ならバージョンアップペース落としても良さそうなのにそれはやらないんだろうな。

## 終わり
Advent Calendarっぽく25個は書けた。ネタならいくらでもあるけどやはり後から修正しにくいQiitaには書きにくい。
