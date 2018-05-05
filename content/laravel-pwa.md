---
title: "LaravelでProgressive Web App(PWA)"
date: 2018-05-05T11:54:02+09:00
categories: ["Laravel"]
draft: false
---

## 前提
SPAではないLaravel5.6+Vue.jsなプロジェクト。

動作確認はPCとAndroidのChrome。Lighthouse。

誰かが作ったライブラリを使うから楽だけど先に技術的な仕組みの概要くらいは調べたほうが良い。

## Service Worker登録とホーム画面に追加
まずは簡単な所から。
用意するファイルは3つほど。

### public/manifest.json
https://developers.google.com/web/fundamentals/web-app-manifest/?hl=ja

もちろんアイコンも必要。

```json
{
  "short_name": "AirHorner",
  "name": "Kinlan's AirHorner of Infamy",
  "icons": [
    {
      "src": "launcher-icon-1x.png",
      "type": "image/png",
      "sizes": "48x48"
    },
    {
      "src": "launcher-icon-2x.png",
      "type": "image/png",
      "sizes": "96x96"
    },
    {
      "src": "launcher-icon-4x.png",
      "type": "image/png",
      "sizes": "192x192"
    }
  ],
  "start_url": "/"
}
```

viewで
```html
<link rel="manifest" href="/manifest.json">
```

### public/sw.js
中身は何もないけどfetchがないとホームに追加は動かないかも。

```
self.addEventListener('install', function (e) {
  console.log('ServiceWorker install')
})

self.addEventListener('activate', function (e) {
  console.log('ServiceWorker activate')
})

self.addEventListener('fetch', function (event) {})
```

### resources/assets/js/app.js
ServiceWorkerの登録。

```
window.addEventListener('load', () => {
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js').
      then(() => {
        console.log('ServiceWorker registered')
      }).
      catch((error) => {
        console.warn('ServiceWorker error', error)
      })
  }
})
```

どこかからコピペしたようなコードだけでホームに追加まではできる。現状ではAndroidくらいでしか有効活用できないと思うけど。

## ウェブプッシュ通知
Laravelの「通知」機能でウェブプッシュ通知が使えるので便利。
他の送信先と同時に通知したりできる。
これ使わずプッシュ通知だけやれと言われても面倒すぎてやらないだろう。
https://github.com/laravel-notification-channels/webpush

これを使うのも結構面倒だけどデモを参考に組み込む。
https://github.com/cretueusebiu/laravel-web-push-demo

sw.jsとか
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/public/sw.js
Vue.jsでの有効／無効とか
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/resources/assets/js/components/NotificationsDemo.vue
分かりにくいけど大体はそのまま使って自分のプロジェクト用に少し書き換えれば済む。

テスト用の通知作っておいたほうのが良かった。事前準備さえできていればLaravelの通知としてシンプルに使える。

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;

use NotificationChannels\WebPush\WebPushMessage;
use NotificationChannels\WebPush\WebPushChannel;

class WebPushTest extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return [WebPushChannel::class];
    }

    /**
     * @param $notifiable
     * @param $notification
     *
     * @return mixed
     */
    public function toWebPush($notifiable, $notification)
    {
        return (new WebPushMessage)->title('WebPush')
                                   ->body('test');
    }
}
```

（当たり前にキュー使ってるけどこういう通知+キューをLaravel以外で作るとどれだけ無駄な手間がかかるか使ってない人は分からない）


次に進む場合sw.jsは`webpush.js`に変更して`resources/assets/js/webpush.js`に置く。

### オフライン
これを使う。
https://github.com/NekR/offline-plugin

```
yarn add offline-plugin -D
```

### webpack.mix.js
cacheはwebpackが生成したファイルを自動で追加するので指定なしでも十分。
externalsでその他のページ。とりあえずトップページ。
SPAかどうかに関わらず完全にオフラインで使えるサイトはあまり作らないので最低限の対応のみ。
entryでwebpush.jsを追加。これでビルドするとsw.jsが生成されるけどそこにwebpush.jsの中身も含まれる。

```
const OfflinePlugin = require("offline-plugin");

mix.webpackConfig({
  plugins: [
    new OfflinePlugin({
      externals: ["/"],
      ServiceWorker: {
        entry: "./resources/assets/js/webpush.js",
        navigateFallbackURL: "/",
        events: true,
        minify: true
      }
    })
  ]
});
```

### resources/assets/js/app.js
app.jsに追加。bootstrap.jsでもいいけど。

```
import * as OfflinePluginRuntime from "offline-plugin/runtime";
OfflinePluginRuntime.install();
```

`navigator.serviceWorker.register('/sw.js')`はこれで行われるので自分で書いてた場合は消す。

Chromeでオフラインにしても表示できれば成功。

## Server PushとServiceWorker
Server Pushは初回の表示を速く。Chromeでは`Push`。
ServiceWorkerでキャッシュすると2回目の表示が速くなる。Chromeでは`from ServiceWorker`。
両方併用すれば良さそう。というか今後はこのくらい普通になっていく。

## 終わり
自分で触って記事にまとめたらService WorkerとPWAが少し分かってきた。
バックグラウンド同期は使いそうにないので試してない。

活かすならオフラインで動くものを作るしかないけどwebでもPCアプリでもオンライン前提のものばかり作ってきたので難しい。
