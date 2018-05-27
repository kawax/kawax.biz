---
title: "Laravel Managerの例としてAuthorizeManagerを作る"
date: 2018-05-26T13:22:48+09:00
categories: ["Laravel"]
draft: false
---

5.0の頃にはあったこのドキュメントが今はなくなってる  
https://readouble.com/laravel/5.0/ja/extending.html

今の各コンポーネントをよく見るとなんとかManagerでも実は`Illuminate\Support\Manager`を使ってない。  
https://github.com/laravel/framework/tree/5.6/src/Illuminate

CacheManagerは5.0で変更されてる。
https://github.com/laravel/framework/blob/4.2/src/Illuminate/Cache/CacheManager.php
https://github.com/laravel/framework/blob/5.0/src/Illuminate/Cache/CacheManager.php

SessionManagerはまだ使ってる。全く変更されてないだけ。
https://github.com/laravel/framework/blob/5.6/src/Illuminate/Session/SessionManager.php

5.6で追加されたLogManagerも違うので最近でも使われてない。

extendやdriverの概念は残ってて使い方は大体同じ。当時どういう方針の変化でこうなってるのかは不明。

よく調べるとLaravel本体では使われなくなってると分かったけど自分で使うなら`Illuminate\Support\Manager`を継承するのが早い。

## 参考にするManager
もちろんSocialite  
https://github.com/laravel/socialite  
例として一番分かりやすいソーシャルログインを早々に公式から提供されたらちょうどいい例がなくて何も作れない。

## AuthorizeManager
今回思い付いたAuthorizeManagerはこの前見つけたGoogle API Clientのauthorize()が元。
https://github.com/google/google-api-php-client/blob/ceb9e53f70bf16a39f7334f7910ac8c5180f1760/src/Google/Client.php#L342

認証済HTTPクライアントを返すManager。  
普通にフォームからログインするようなサイトが主な対象。  
スクレイピングの前段階として使える。  
最近は主要なサイトはAPI提供してるしログインした上でのスクレイピングなんてほとんどしないけど、少ないからこそ例としてはちょうどいい。

- APIがあるならAPIを使うべき。
- 相手サーバーの負荷を考えない過度なスクレイピングは避けるべき。

## 基本的な使い方
先にこういう使い方ができるようにしたいという例。

```php
    $credentials = [
        'mail'     => '',
        'password' => '',
    ];

    if (Authorize::driver('sample')->login($credentials)) {
        /**
         * @var \Goutte\Client $client
         */
        $client = Authorize::driver('sample')->client();
        dump($client);
    }
```

ログイン済Goutte\Clientが得られるので後はGoutteの仕事。プロジェクト内で使うならスクレイピング部分までDriverに含めるけどパッケージならなるべくシンプルな単機能に徹する。

## 初期のAuthorizeManager
Managerクラスとして必須なのは`getDefaultDriver()`のみ。
https://github.com/laravel/framework/blob/3976a4388dc80272ed7e18b999e64757756116eb/src/Illuminate/Support/Manager.php

DefaultDriverがないならSocialiteのように例外を発生させればいい。AuthorizeManagerではただのGuzzleHttp\Clientを返すDefaultDriverを用意している。

```php
namespace Revolution\Authorize;

use Illuminate\Support\Manager;

use Revolution\Authorize\Contracts\Factory;

class AuthorizeManager extends Manager implements Factory
{
    /**
     * Get the default driver name.
     *
     * @return string
     */
    public function getDefaultDriver()
    {
        return 'default';
    }

    /**
     * Create an instance of the specified driver.
     *
     * @return \Revolution\Authorize\Drivers\AbstractDriver
     */
    protected function createDefaultDriver()
    {
        return new Drivers\DefaultDriver();
    }
}
```

ここからDriverを増やしていく。

## Driver Interface
型とかは厳密には決めてない。ログインに必要な情報はサイトごとに違うし、何のClientを返すかもDriverの自由。
普通はGoutteかGuzzleだろうけど別にGoogle_Clientでもいい。

```php
interface Driver
{
    /**
     * Login.
     *
     * @param mixed $credentials
     *
     * @return bool
     */
    public function login($credentials = null): bool;

    /**
     * Client.
     *
     * @return mixed
     */
    public function client();
}
```

$credentialsはarrayでもobjectでも。

```php
    $credentials = [
        'mail'     => '',
        'password' => '',
    ];
```

```php
    $credentials = new Credentials([
        'mail'     => '',
        'password' => '',
    ]);
```

Credentialsクラスも用意してるけどDriver内ではdata_get()で見てるので本当にどっちでも同じ。
独自のDriverでは独自のクラスを使ってもいい。

## Drivers
ログイン方法が分かるサイトをいくつか選んだだけ。

### DefaultDriver
なにもせずGuzzleHttp\Client返すだけのdefault。

```php
namespace Revolution\Authorize\Drivers;

use GuzzleHttp\Client;

class DefaultDriver extends AbstractDriver
{
    /**
     * @var Client
     */
    private $client;

    /**
     * DefaultDriver constructor.
     */
    public function __construct()
    {
        $this->client = new Client(['cookies' => true]);
    }

    /**
     * Login.
     *
     * @param mixed $credentials
     *
     * @return bool
     */
    public function login($credentials = null): bool
    {
        return true;
    }

    /**
     * Client.
     *
     * @return mixed
     */
    public function client()
    {
        return $this->client;
    }
}
```

Driverを作ってAuthorizeManagerに`create〇〇Driver()`を増やす作業の繰り返し。

### NiconicoDriver
Goutteで普通にPOSTする例。

```php
public function login($credentials = null): bool
{
    $crawler = $this->client->request('POST', 'https://account.nicovideo.jp/api/v1/login', [
        'mail_tel' => data_get($credentials, 'mail'),
        'password' => data_get($credentials, 'password'),
    ]);

    return $crawler->getUri() === 'https://account.nicovideo.jp/my/account';
}
```

### A8netDriver
Goutteのformを使う例。

```php
public function login($credentials = null): bool
{
    $crawler = $this->client->request('GET', 'https://www.a8.net/');

    $form = $crawler->filter('form[name=asLogin]')->form();

    $crawler = $this->client->submit($form, [
        'login'  => data_get($credentials, 'login'),
        'passwd' => data_get($credentials, 'password'),
        'moa'    => '/a8',
    ]);

    return $crawler->getUri() === 'https://pub.a8.net/a8v2/asMemberAction.do';
}
```

### ValueCommerceDriver
特定のCookieが必要なちょっと特殊な例。

```php
use Goutte\Client;
use Symfony\Component\BrowserKit\Cookie;

public function login($credentials = null): bool
{
    $crawler = $this->client->request('GET', 'https://aff.valuecommerce.ne.jp/?type=4');

    //これがないとJSかCookieが無効と判断される
    $this->client->getCookieJar()
                 ->set(new Cookie('I_do_Javascript', 'yes', strtotime('+1 day')));

    $form = $crawler->filter('form')->form();

    $crawler = $this->client->submit($form, [
        'login_form[emailAddress]'    => data_get($credentials, 'mail'),
        'login_form[encryptedPasswd]' => data_get($credentials, 'password'),
    ]);

    return $crawler->getUri() === 'https://aff.valuecommerce.ne.jp/home';
}
```

### その他
Goutteでログインできるなら簡単。
最近はそれじゃログインできないサイトも多いのでそういう場合はheadless chromeとかが必要かも。

node.jsでのheadless chromeは前に試した。
https://github.com/kawax/headless-chrome-google-login

当時はPHP用はなかったけど今は見つけられるのでこういうのでなんとかなるかも（未確認）
https://github.com/chrome-php/headless-chromium-php
あくまでもClientとして何を返すかは自由なのでDriver次第。

## 拡張
Socialiteと同じくextend()で好きなように拡張できる。

### プロジェクト内でのみ使う場合
どこかにCustomDriverを作る。
app/Authorize/CustomDriver.php

AppServiceProviderで登録。

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;

use Revolution\Authorize\Facades\Authorize;
use App\Authorize\CustomDriver;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Authorize::extend('custom', function ($app) {
            return new CustomDriver;
        });
    }
}
```

customとして使えるようになる。

```php
if (Authorize::driver('custom')->login($credentials)) {
    /**
     * @var \GuzzleHttp\Client $client
     */
    $client = Authorize::driver('custom')->client();
}
```

### composerパッケージにする場合
ServiceProviderを作って…とSocialiteと同じなので省略。

Google_ClientのDriver作ったのでこれを参考に。  
https://github.com/kawax/authorize-google-api

### AuthorizeManager本体に入れたい場合
GitHubでプルリク。

## Laravel以外で使う
Laravel ManagerではあるけどcomposerでインストールすればLaravel以外でも使えるはず。

```php
    use Revolution\Authorize\AuthorizeManager;

    $manager = new AuthorizeManager(null);

    if($manager->driver()->login()){
        $client = $manager->driver()->client();
    }
```

## 終わり
この説明書くのが一番大変なのでこの辺で一旦公開…。

- https://github.com/kawax/authorize
- https://github.com/kawax/authorize-project
