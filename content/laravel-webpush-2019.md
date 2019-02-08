---
title: "LaravelでWeb Push 2019"
date: 2019-02-07T17:13:57+09:00
categories: ["Laravel"]
draft: false
---

去年も書いたけど2019年版。WebPushに関しては何も変わってない。

## 環境
- Mac
- Laravel 5.7
- PHP 7.2
- Chrome

## プロジェクト作成
いつものとパッケージのインストール。
https://github.com/laravel-notification-channels/webpush

```
laravel new laravel-webpush-project && cd $_
php artisan make:auth
composer require laravel-notification-channels/webpush
composer require laravel/homestead --dev
php vendor/bin/homestead make
```

WebPush(正確にはService Worker)をローカルで動かすには保護されたhttpsかlocalhostで動かす必要がある。
今回はlocalhostを使用。
http://localhost:8000/

書いてる時点ではPHP7.3+xdebug betaはバグがあるのでPHP7.2。

普通のLaravel通りユーザー登録してhomeの表示まで確認。

## ext-gmp
サーバー環境で重要な点としてgmp PHP拡張が必須。

Homesteadなら

```
sudo apt-get install php-gmp
```

でインストール。

本番環境でも忘れないようにする。

## WebPush Notification
Laravel5.5以上なのでServiceProviderの追加は不要。

### UserモデルにHasPushSubscriptions追加。

```php
use NotificationChannels\WebPush\HasPushSubscriptions;

class User extends Model
{
    use HasPushSubscriptions;
}
```

### migrationとconfigの公開。
```
php artisan vendor:publish
```

`NotificationChannels\WebPush\WebPushServiceProvider`の数字を選択。

migrate実行。

```
php artisan migrate
```

configは触らなくていい。

### VAPID
```
php artisan webpush:vapid
```

`.env`が自動で変更される。

Firebase使うような記事があるけどきちんと最新情報を追ったほうがいい。去年書いた時点でもVAPIDの仕組みができていたのでFirebase不要で無料で使える。
https://qiita.com/tomoyukilabs/items/9346eb44b5a48b294762
https://developers.google.com/web/fundamentals/push-notifications/web-push-protocol

（実態としてはWebPush普及のためにGoogleやMozillaが無料でサーバーを提供している）

## JavaScript側
Laravel側は簡単なので今回はJS側から。

まずデモプロジェクトをダウンロードしておく。必要なファイルはここからコピペすればいい。
https://github.com/cretueusebiu/laravel-web-push-demo

以下はなるべくデモから変えない方針でのやり方。

### sw.js
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/public/sw.js
`public/sw.js`に保存。変更不要。
Service Worker。使うだけなら細かいことは気にしなくていい。

### NotificationsDemo.vue
WebPushを有効化したりテスト通知を送るボタン。
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/resources/js/components/NotificationsDemo.vue
`resources/js/components/NotificationsDemo.vue`に保存。変更不要。
WebPushのメイン部分だけどこれも使うだけならscript部分は気にしなくていい。


app.jsにコンポーネント追加。

```
Vue.component('notification-component', require('./components/NotificationsDemo.vue').default);
```

JSのビルド

```
yarn install
yarn prod
```

### viewの変更
home.blade.phpに追加。

```
<notification-component></notification-component>
```

app.blade.phpに追加。VAPIDをVue.jsに渡すために必要。
vapidPublicKeyのみでいいのでデモからは減らしている。

```
<script>
    window.Laravel = {!! json_encode([
        'vapidPublicKey' => config('webpush.vapid.public_key'),
    ]) !!};
</script>
```

https://github.com/cretueusebiu/laravel-web-push-demo/blob/48082debae16682602068c5908ff18bc36985dfc/resources/views/layouts/app.blade.php#L22

### notification-icon.png
`public/notification-icon.png`にコピー。ただのアイコンなのでなくてもいい。
https://github.com/cretueusebiu/laravel-web-push-demo/tree/master/public


ここまででSend NotificationとEnable Push Notificationsのボタンが表示されてプッシュ通知の許可ダイアログまでは動く。
Laravel側の準備ができてないのでそれ以上は動かない。

## Laravel側

### Notifications
`app/Notifications/HelloNotification.php`にコピー。
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/app/Notifications/HelloNotification.php

WebPush以外の通知先は削除。

```php
public function via($notifiable)
{
    return [WebPushChannel::class];
}
```

デモはEventsも使ってるけど不要。

### コントローラー
NotificationController.phpとPushSubscriptionController.phpをコピー。
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/app/Http/Controllers/NotificationController.php
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/app/Http/Controllers/PushSubscriptionController.php

NotificationControllerのEventsは不要なので削除。
WebPushのみであればstore()しか使ってない。

### ルーティング
`// Notifications`以下をコピー。VAPIDなのでmanifest.json部分は不要。
https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/routes/web.php

## 動作確認
ここまででWebPushの通知は可能。

Enable Push Notificationsで通知の許可。
許可後はDisableに変わる。

Send Notificationで送信。
通知が表示されれば成功。

WebPushだけならPWAの説明で出てくるようなことは無視しても動かせる。

## 複数ユーザーへの通知
コントローラーからとartisanコマンドから

```
php artisan make:controller SendUsersController
php artisan make:command SendUsersCommand
```

どっちでもユーザーを指定するだけ。普通のLaravelの通知機能の範囲。アプリ内のどこからでも送信可能。

```php
use Illuminate\Support\Facades\Notification;
use App\Notifications\HelloNotification;
use App\User;

Notification::send(User::all(), new HelloNotification());
```

もちろん条件指定してもいい。

```php
$users = User::where()->...
Notification::send($users, new HelloNotification());
```

NotificationsDemo.vueにSend all usersボタンを増やす。
ボタンとsendUsersNotificationをコピペで増やして少し修正するだけ。

artisanコマンドからは`php artisan webpush:users`

## サンプル以上の使い方
WebPushを試すだけならここまででいいけど実際のアプリではボタンから送信なんてことはしないだろう。

使うのはsw.jsとNotificationsDemo.vueの有効化／無効化の部分。
sw.jsはそのまま。
NotificationsDemo.vueはSendボタンは削除。名前も適当なものに変更。
WebPush.vueがSendを削除した版。script部分は変更しなくていい。変更するとしてもtemplateだけなので簡単。
WebPush.vueを設定ページなどに表示してユーザーに許可してもらう。

WebPush初期はサイトを表示しただけで許可ダイアログが出るようなスパム的な使い方が多かったせいで嫌われてる。
ブラウザ側が対応してもうできなくなるはずなので必ずユーザーからのアクションで許可ダイアログを出す。

送信は同じ。

### ルーティングを変えたい場合
`/notifications`や`/subscriptions`のルーティングを変えるならNotificationsDemo.vueの修正も必要。（sw.jsのdismiss部分は使われてないので削除してもいい）

例えば`/push/notifications`にしたら

```php
Route::prefix('push')->group(function () {
  Route::post('notifications', 'NotificationController@store');
});
```

Vue.jsも変更。

```
axios.post('/push/notifications')
```


## WorkBox
去年の段階ではなかったものとしてはWorkBoxがあるけどこれはWebPushには使えない。
https://developers.google.com/web/tools/workbox/
Google製なのでWebPush以外のPWA機能も使うなら検討。


## そもそもの話
WebPushはまだiOSが対応してない。プッシュ通知したいだけならチャット使ったほうが楽だったりする。
プッシュ通知のためだけにスマホアプリ作ろうとしてる人を未だに見かけるけどそれは何年も前の発想。
スマホアプリをインストールしてくれるようなユーザーがすでにいるならチャットに誘導したほうが早い。

## 終わり
https://github.com/kawax/laravel-webpush-project
