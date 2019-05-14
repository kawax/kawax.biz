---
title: "[WIP]テストしにくいコード"
date: 2019-05-13T11:26:38+09:00
categories: ["Laravel"]
draft: false
---

## 対象
LaravelとPHP

## new
よく言われている分かりやすい目印。`new`使ってるだけでやばいなと分かる。

```php
$foo = new Foo();
```

LaravelならDIかapp()/resolve()でテスト時にモックしやすい実装にできる。

```php
public function __invoke(Foo $foo)
{
    $foo->bar();
}
```

```php
$foo = resolve(Foo::class);
```

テストではこうやってモック。

```php
$this->mock(Foo::class, function ($mock) {
    $mock->shouldReceive('bar')->once();
});
```

Laravelでの例外。Mail、Event、Notificationなどの`new`は簡単にモックできるので使ってOK。

https://readouble.com/laravel/5.8/ja/mocking.html

## staticメソッド
あまりにも有名すぎて自分では使わないから知らなかったけど世の中には本当にstaticメソッド使いまくる人が存在する。
staticメソッド使っていいのはそのメソッド単体で完結していて他への影響がない場合に限る。
staticメソッド内でDB操作なんかしていたらとんでもなくテストしにくいコード。
他人が書いたstaticメソッドをテストしようとするとstaticメソッドやめろと言われてる理由が理解できる。

```php
$bar = Foo::bar();
```

こう使えばいいだけのことなのでstaticにするメリットは全く分からない。（一般的ではないけど実はstaticメソッドでもこう書ける。）

```php
$bar = (new Foo)->bar();
```

staticじゃないだけでモックできる実装に。（実装のほうを変えればstaticでもモックできるかもしれないけどそれができるならstaticじゃなくすこともできる）

```php
$bar = resolve(Foo::class)->bar();
```

LaravelでstaticメソッドはStrとかArr。
Facadeはstaticに見えてstaticじゃないしテストできるので問題ない。

## それでもテストしにくい
Guzzleの非同期通信とかReactPHPとかのPromiseはどうテストすればいいのか未だに分かってない。
http://docs.guzzlephp.org/en/stable/quickstart.html

## 参考
Laravel関連で唯一役に立った本。

Laravel Testing Decoded
https://leanpub.com/laravel-testing-decoded
日本語版
https://leanpub.com/laravel-testing-decoded-japanese

Laravel4時代なので今から読んでどうなのかは分からないけど。

Laravelの使い方はドキュメント読めば分かる。
良いコードや設計はLaravelの本読んでも意味がない。フレームワークや言語には依存しない話。
