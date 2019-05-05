---
title: "Laravel + Homestead + PhpStorm でテストカバレッジの表示"
date: 2019-05-05T12:06:36+09:00
categories: ["Laravel"]
draft: false
---

## 環境
- Laravel 5.8
- Homestead box 7.2.1
- Xdebug 2.7.1
- VirtualBox 5.2。6.0は2回目以降の起動でエラー出るのでまだ使わないほうがいいかも。
- PhpStorm 2019.1。コードスタイルのプリセットにLaravelが追加されたので最近はLaravelプリセットに合わせてる。PSR-2+Laravel独自部分。

## プロジェクト準備
Homesteadはプロジェクト毎にインストール。

```
laravel new phpstorm-coverage && cd $_
composer require laravel/homestead --dev

//Mac / Linux:
php vendor/bin/homestead make
//Windows:
vendor\bin\homestead make

//Homestead.yaml編集。box7.2.1以降はhostnameの設定も大事。

vagrant up
```

`hostname: phpstorm-coverage`ならすでに`http://phpstorm-coverage.local/`で表示できるはず。できなかったら一旦`vagrant reload`
Macなら何もいらない。Windowsは10なら対応してるとかしてないとか情報がはっきりしないので保留。

box7.2.1でavahiがインストールされるようになったのでhostsの設定不要で使えるようになった。
VirtualBox6.0ではエラーっぽいので5.2で。

## テストの準備

```
php artisan make:controller WelcomeController -i
```

テスト対象のコードは別になんでもいい。

```php
public function __invoke(Request $request)
{
    if ($request->filled('test')) {
        $test = $request->input('test');
    } else {
        $test = 'test';
    }

    return view('welcome')->with(compact('test'));
}
```

テストも適当に。コントローラーの`$test = 'test';`は通らない。

```php
public function testBasicTest()
{
    $response = $this->get('/?test=a');

    $response->assertStatus(200)
             ->assertViewHas('test', 'a');
}
```

## PhpStormの設定
日本語のドキュメントがあるので詳細は省略。

### リモートインタープリター
まずVagrant内のPHPで実行するように設定。
https://pleiades.io/help/phpstorm/php-interpreters.html
https://pleiades.io/help/phpstorm/configure-remote-language-interpreter.html

- 「...」からCLI Interpreters
- 「＋」から`From Docker, Vagrant, VM, Remote...`
- Vagrantを選択

Vagrantが正しく起動していればPHPのpath等が表示される。

### オンデマンドモードでXdebug有効化
「CLI Interpreters」画面の「Debugger extension」に`/usr/lib/php/20180731/xdebug.so`。
https://pleiades.io/help/phpstorm/configuring-xdebug.html

box7.2.1時点ではPHP7.3とXdebugのバグのせいかデフォルトでは無効。
別バージョンで元から有効ならここは不要。

### テスト・フレームワーク設定
リモートPHPインタープリターを使うので手動での設定が必要。
https://pleiades.io/help/phpstorm/using-phpunit-framework.html

### テスト構成
Vagrant内のPHPを使わない場合と同じ。
https://pleiades.io/help/phpstorm/creating-run-debug-configuration-for-tests.html

## カバレッジ付きで実行
ここまで準備できていればテスト実行してカバレッジが表示される。エディターでは実行された行毎に色分けされる。一応テストされて問題ないはずの行が分かる。
https://pleiades.io/help/phpstorm/running-test-with-coverage.html

## 終わり
自分で一から作ったプロジェクトなら必要ないけど他人のプロジェクトを見るなら必要。
テストもないようなプロジェクトならまずテスト書いてカバレッジで問題ない範囲を修正していくことになる。
