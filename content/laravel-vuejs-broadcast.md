---
title: "Laravel + Vue.js ブロードキャスト通知"
date: 2018-12-15T12:03:51+09:00
categories: ["Laravel"]
draft: false
---

前回の続きでブロードキャスト通知部分を実際に作ってみる。
https://kawax.biz/laravel-vuejs-tutorial/

## 前提
入門レベルではなくVue.jsを使うと何がいいのかという記事。

ローカルで動かすまで。本番環境で動かす方法は説明が長くなるので省略。

## 準備
Laravel側もしっかり作るのでここからは開発環境にHomesteadを使う。

最近リリースされたLaravel WebSocketsもインストール。
https://github.com/beyondcode/laravel-websockets
https://docs.beyondco.de/laravel-websockets/
詳しくはLaravel WebSocketsのドキュメント参照。
```
composer require beyondcode/laravel-websockets
php artisan vendor:publish
# BeyondCode\LaravelWebSockets\...を選択

php artisan migrate
```

キュードライバーにRedis使うためにpredisも。

```
composer require predis/predis
```

npmに追加。

```
npm i laravel-echo pusher-js -D
```

### .env
Pusher AppのIDなどはなんでもいい。

```
BROADCAST_DRIVER=pusher

QUEUE_CONNECTION=redis

PUSHER_APP_ID=test_app
PUSHER_APP_KEY=test_key
PUSHER_APP_SECRET=test_secret
```

## 通知を作る
```
php artisan make:notification TestNotification
```

messageを送るだけの簡単な通知。

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Messages\MailMessage;

use Illuminate\Notifications\Messages\BroadcastMessage;

class TestNotification extends Notification implements ShouldQueue
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
     * @param  mixed $notifiable
     *
     * @return array
     */
    public function via($notifiable)
    {
        return ['broadcast'];
    }

    /**
     * @param  mixed $notifiable
     *
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'message' => 'message ' . str_random(6),
        ]);
    }
}
```

### 通知を送るAPI
routes/web.php

```php
Route::middleware('auth')->group(function () {
    Route::post('api/send', 'SendController');
});
```

SendController

```php
use App\Notifications\TestNotification;

class SendController extends Controller
{
    public function __invoke(Request $request)
    {
        $request->user()->notify(new TestNotification);

        return [
            'status' => 'OK',
        ];
    }
}
```

## Vue.js側
通知送信用のボタン
SendComponent.vue
postしてるだけ。

```
<template>
    <button type="button" class="btn btn-primary" @click="send">Send Notification</button>
</template>

<script>
    export default {
        methods: {
            send () {
                const res = axios.post('/api/send')
            },
        },
    }
</script>
```

NotificationsComponentも変更。

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
        props: [
            'userId',
        ],
        mounted () {
            this.getNotifications()
            this.echo()
        },
        methods: {
            async getNotifications () {
                const res = await axios.get('/api/notifications')
                if (res.status === 200) {
                    this.notifications = res.data
                    console.log(this.notifications)
                }
            },
            echo () {
                Echo.private('App.User.' + this.userId).notification((notification) => {
                    this.notifications.unshift(notification)
                    console.log(notification)
                })
            },
        },
    }
</script>
```

ドキュメントでは`Echo.private('App.User.' + userId)`だけで分かりにくいけど`userId`は自分で設定する。
https://readouble.com/laravel/5.7/ja/notifications.html#broadcast-notifications
Vueコンポーネントのpropsで`userId`

```
props: [
    'userId',
],
```

Blade側でuser-idを渡す。

```
<notifications-component user-id="{{ auth()->user()->id }}"></notifications-component>
```

これでVueコンポーネント内のuserIdが設定されるので
`Echo.private('App.User.' + this.userId)`で受け取れる。

app.jsで登録とbootstrap.jsのEcho部分も変更。

```
import Echo from 'laravel-echo'

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    encrypted: false,
    wsHost: window.location.hostname,
    wsPort: 6001,
    disableStats: true,
});
```

## キューワーカーとWebSocketサーバーの起動
ここまで作ったものの処理の流れをまとめると
1. Vue.jsのSendComponentをクリックで/api/sendにpost
2. Laravel側で受け取ってTestNotificationからブロードキャスト通知を実行

次に必要なのはブロードキャスト通知を処理する部分。
ブロードキャストにはキューが必要なのでついでにキューも。

詳しくはドキュメント参照。
https://readouble.com/laravel/5.7/ja/broadcasting.html

ドライバーはpusherとredisがありPusherを使うなら本来は https://pusher.com/ への登録が必要だけどLaravel WebSocketsはPusherの代わりとして使える。
なので次はWebSocketsサーバーの起動。
設定さえ間違えてなければ起動するだけで完了。

config/broadcasting.php
```
'pusher' => [
    'driver' => 'pusher',
    'key' => env('PUSHER_APP_KEY'),
    'secret' => env('PUSHER_APP_SECRET'),
    'app_id' => env('PUSHER_APP_ID'),
    'options' => [
        'cluster' => env('PUSHER_APP_CLUSTER'),
        'encrypted' => false,
        'host' => '127.0.0.1',
        'port' => 6001,
        'scheme' => 'http'
    ],
],
```

重要なのはvagrant内で実行すること。2つ実行するのでタブを分けるなどして2つ同時に実行したままにする。

```
vagrant ssh
cd code/
php artisan websockets:serve
```

```
vagrant ssh
cd code/
php artisan queue:work
```

ブラウザからSend Notificationをクリックして通知が増えるか動作確認。上手く動かないならどこか間違えてるので修正。Laravelを修正した場合は両方の再起動が必要。

処理の流れの続きとしては

3. WebSocketsサーバーがブロードキャスト通知をブラウザ側に送信
4. NotificationsComponentが受け取って表示

## supervisorで自動起動
手動でWebSocketsサーバーの起動は面倒なので自動化。
Homesteadをインストールするとafter.shができているのでこれを変更。

```
laravel_worker_block="[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/vagrant/code/artisan queue:work
autostart=true
autorestart=true
user=vagrant
numprocs=1
redirect_stderr=true
stdout_logfile=/home/vagrant/code/storage/logs/queue.log"

laravel_websockets_block="[program:laravel-websockets]
process_name=%(program_name)s_%(process_num)02d
command=php /home/vagrant/code/artisan websockets:serve
autostart=true
autorestart=true
user=vagrant
numprocs=1
redirect_stderr=true
stdout_logfile=/home/vagrant/code/storage/logs/websockets.log"

sudo sh -c "echo '$laravel_worker_block' > '/etc/supervisor/conf.d/laravel-worker.conf'"
sudo sh -c "echo '$laravel_websockets_block' > '/etc/supervisor/conf.d/laravel-websockets.conf'"

sudo supervisorctl reread
sudo supervisorctl update
```

`vagrant provision`でプロビジョンし直せば起動した状態になる。修正後の再起動や、vagrant up後に動いてない時も`vagrant provision`してみる。この辺りは自分で経験積んで学ぶしかない。

## artisanコマンドから通知
通知さえ送ればなんでもいいのでartisanコマンドからでも。

```
public function handle()
{
    User::first()->notify(new TestNotification);
}
```

artisanコマンドを実行すればブラウザ側の通知が自動で増えてることが確認できる。
このコマンドもvagrant内で実行しないと失敗するはず。

## まとめ
ブラウザでクリックしたりartisanコマンドだったり手動で実行を例にしてるけど実際はなんでもいい。
タスクスケジュールでartisanコマンドを定期実行してその結果を通知など。

こういうリアルタイム通知はVue.jsというかJavaScript使わないと不可能。
Laravelだけでできることを無理にVue.jsに置き換えてる人をよく見るけどそんなのよりブロードキャスト通知使ってみるとVue.jsでしかできないことがあると理解できる。

https://github.com/kawax/laravel-vuejs-tutorial/tree/broadcast
