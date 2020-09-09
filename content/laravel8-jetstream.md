---
title: "Laravel8 Jetstreamの調査"
date: 2020-09-05T20:35:18+09:00
categories: ["Laravel"]
draft: false
---

Jetstreamを試すためにLaravel8リリース直前の段階で新規プロジェクト作って調べてみた。
https://github.com/laravel/jetstream

Livewire版
https://github.com/kawax/laravel8-jetstream
Inertia版。＋チーム機能。
https://github.com/kawax/laravel8-jetstream-inertia

## Jetstream
Laravelの認証機能を含む全部入りのデフォルトプロジェクト。
Laravel5.8までは`make:auth`、Laravel6以降は`laravel/ui`パッケージに分離してたけどLaravel8以降はJetstreamが全部入りの「松」扱いになりそう。

ただしJetstreamはちょっと過剰すぎるとか、LivewireもしくはInertiaを使いたくない場合は旧来の認証機能を使えばいい。

## インストーラーv4以降の新規作成方法
まずインストーラーを最新バージョンに。
```
composer global require laravel/installer
```

### Jetstream（松）
```
laravel new my-project --jet
```

Livewire or Inertiaやチーム機能を使うか聞かれる。

### 旧来の認証機能（竹）
`--auth`オプションは削除されたのでもう使えない。
`laravel/ui`を手動でインストール。

```
laravel new my-project
cd ./my-project
composer require laravel/ui
php artisan ui vue --auth
```

### 認証機能なし（梅）
```
laravel new my-project
```

## プロジェクトの中身の調査
ルーティングもコントローラーもないままある程度の機能が揃ってる。
Jetstreamパッケージ内で定義してるからなので処理の流れを追うのがそこそこ大変。

一番難しくしてる部分はLivewire or Inertiaなので先にこっちの理解が必要。
https://laravel-livewire.com/
https://inertiajs.com/

## Livewire版
Livewire/Inertiaどっちでも機能は同じなのでLivewire版で確認。

LivewireはPHPとBladeでモダンフロントっぽいものを作ることができる。
JavaScript書けなくてもいいのがメリット。
Laravel+Livewireの独自技術に依存しすぎるのがデメリット。

### ルーティング
長いのでテキストファイルに。
https://github.com/kawax/laravel8-jetstream/blob/master/routes.txt

### dashboard
routes/web.php。これは普通。

```php
Route::middleware(['auth', 'verified'])->get('/dashboard', function () {
    return view('dashboard');
})->name('dashboard');
```

dashboard.blade.phpは
```
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            Dashboard
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                <x-jet-welcome />
            </div>
        </div>
    </div>
</x-app-layout>
```

`<x-app-layout>`や`<x-jet-welcome />`が出てきた。これはLaravel7で全面的に作り直されたBladeコンポーネント。
Livewire版はBladeコンポーネントを全力で活用してる上にLivewireの機能も混ざってるので読み解きにくい…。
`<x-app-layout>`の本体は`app/View/Components/AppLayout.php`。`app/View/Components/`内はLaravelが自動的に読み込む。クラス名から`x-app-layout`まで自動的に決まる。
`<x-jet-welcome />`の本体は Jetstreamパッケージ内にある。 https://github.com/laravel/jetstream/blob/master/resources/views/components/welcome.blade.php
JetstreamServiceProviderでコンポーネント登録時に`jet`の接頭辞を付けている。

```php
Blade::component('jetstream::components.'.$component, 'jet-'.$component);
```

dashboard.blade.phpを見て`<x-app-layout>`や`<x-jet-welcome />`はなんだろと調べようとしても本体の場所が違うので探しにくい。
どっちもBladeコンポーネントではあるけど。
Laravel7以降の新しいBladeコンポーネントの理解が必須。

`x-`はBladeコンポーネントの印。
`x-jet-`はJetstream内で定義。
厄介なことにLivewireの使うAlpine.jsでも`x-`が使われるので一部の`x-`は違うと覚えておく必要がある。
https://github.com/alpinejs/alpine
一つのbladeファイル内で両方の`x-`が使われてるので後から読むと辛さしかないな。

調べ方は分かったので終了。

## Inertia版
Livewireと違ってInertiaはView側をVueコンポーネントで作る。

ルーティング→コントローラー→ビュー(Blade)からビューだけをVueに置き換えたような形。
ビューだけ変えてるけどサーバーサイドフレームワークの普通の使い方をするだけで全体としてはSPAが出来上がるのがInertiaの特徴っぽい。

JavaScriptを書きたくないならLivewire。
全部Vueで作りたいならInertia。Bladeの機能は使えないけどほぼ純粋なVueなのでLivewire版程の違和感はない。

## ひとまず終わり
どう使うかはまだ分からない。
LivewireもInertiaもどっちも使いたくない…。

「竹」で十分。
ただし`laravel/ui`も今後は使うなってことなので
https://github.com/laravel/ui
今後の様子見て使ったほうがいい状況になれば使う。

新規作成はJetstream使うけどその後は今まで通りの使い方できるか調べたほうが良さそう。