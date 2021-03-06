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
日本語のドキュメントがあるので詳細は省略。ドキュメント読んでも分かりにくいので実際に触ってみるしかないけど。

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
別バージョンで元から有効ならここは不要。もしくは無効化してからオンデマンドモードの設定。

### テスト・フレームワーク設定
リモートPHPインタープリターを使うので手動での設定が必要。
https://pleiades.io/help/phpstorm/using-phpunit-framework.html

### カバレッジの設定
「アクティブなスイートを新しいものに置き換える」
https://pleiades.io/help/phpstorm/configuring-code-coverage-measurement.html

### テスト構成
Vagrant内のPHPを使わない場合と同じ。
https://pleiades.io/help/phpstorm/creating-run-debug-configuration-for-tests.html

## カバレッジ付きで実行
ここまで準備できていればテスト実行してカバレッジが表示される。エディターでは実行された行毎に色分けされる。一応テストされて問題ないはずの行が分かる。
https://pleiades.io/help/phpstorm/running-test-with-coverage.html

## 終わり
自分で一から作ったプロジェクトではあまり必要ないけど他人のプロジェクトを見るなら必要。
テストもないようなプロジェクトならまずテスト書いてカバレッジで問題ない範囲を修正していくことになる。

## 追記 Travis+Code Climateで表示
ローカルでの表示はいらないけど外部サービスを使って表示する場合。

https://docs.codeclimate.com/docs/travis-ci-test-coverage

1. Code Climateで`TEST REPORTER ID`を調べる。
2. TravisのEnvironment Variablesで`CC_TEST_REPORTER_ID`を設定。
3. 以下をそれぞれ追加

.travis.yml

```yml
before_script:
  - curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./cc-test-reporter
  - chmod +x ./cc-test-reporter
  - ./cc-test-reporter before-build

after_script:
  - ./cc-test-reporter after-build -t clover --exit-code $TRAVIS_TEST_RESULT
```

phpunit.xml

```xml
<logging>
    <log type="coverage-clover" target="build/logs/clover.xml"/>
</logging>
```

## 追記2 MacローカルのPHPでXdebugの有効化
Vagrantの起動が必須なのも面倒なのでローカルPHPでも可能にする。

https://xdebug.org/docs/install

PHPはbrewでインストールでもxdebugはpecl使えということなので

```
pecl install xdebug
```

エラー出る場合は検索。

最後にここまで出れば成功。
```
Extension xdebug enabled in php.ini
```

ただしローカルのPHPで直接有効化はしない。
普通にインストール成功した場合php.iniの一番上に`zend_extension=xdebug.so`が追加されてるのでまずこれを消す。

↑で書いたPhpStormのオンデマンドモードでのみ有効化と同じように「Debugger extension」を設定。
`/usr/local/Cellar/php/`以下辺りを探す。

これでカバレッジ付きでテストを実行した時のみXdebugが有効。普段のartisanやcomposer updateなどが遅くならない。

昔Xdebug使った時は常に有効で遅くなってたけどこれならVagrant使わなくてもいいかも。