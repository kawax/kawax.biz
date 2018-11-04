---
title: "Laravel で Discord API"
date: 2018-10-31T10:49:57+09:00
categories: ["Laravel"]
draft: false
---

一応ゲーマー向けチャットのはずだけどいつの間にかLaravelやVue.jsの公式チャットもDiscordに移行していた。

- https://laravel.com/docs/5.7/contributions
- https://vue-land.js.org/

Reactがメンバー多すぎてDiscordに移行したのが最初？

- https://reactjs.org/blog/2015/10/19/reactiflux-is-moving-to-discord.html

APIを色々試してみる。  

- https://github.com/kawax/discord-project

## テスト用チャット
誰でも参加可能。ボイスチャットは不可。Laravelチャンネルも作ったので人が増えたら何か自動で情報流すかもしれないけど増えるとは思ってない。

- https://discord.gg/XukjEsV

<iframe src="https://discordapp.com/widget?id=506978290934874132&theme=dark" width="350" height="500" allowtransparency="true" frameborder="0"></iframe>

## 環境
- Laravel 5.7
- Forge
- ローカルサーバーはなし。artisanコマンドしか使ってない。

## アプリケーションとBot
作り方は他所にいくらでも書いてあるのでここでは書かない。

- https://discordapp.com/developers/docs/intro

CLIENT IDやBotのTOKENを控えておく。

## メッセージ送信制限
一度だけWebSocketで接続する必要がある。

- https://discordapp.com/developers/docs/resources/channel#create-message

## ライブラリ
WebSocketと非同期なDiscord APIはPHPには向いてない。どれが一番メジャーなのかは知らないけどおそらくnode.jsが向いてる。

- https://discord.js.org/

リアルタイムなやり取りとか苦手なことはひとまず置いておいてLaravelが得意なことをやる。

DiscordPHPは死んでる。

- https://github.com/teamreflex/DiscordPHP

DiscordPHPに依存してるBotManも停止したまま。

- https://github.com/botman/driver-discord

PHPでもnode.jsのように使うならたぶんこれ。

- https://github.com/CharlotteDunois/Yasmin

基本的なREST API

- https://github.com/restcord/restcord

最終的に一番参考になったのはなんとLaravel Notificationsだった。サーバーへのBotの追加やWebSocketへの接続を`php artisan discord:setup`一つで解決してるのが一番スマート。

- https://github.com/laravel-notification-channels/discord

Laravel NotificationsとRestCordを使う。

## Notificationsのメッセージ送信まで

```
composer require laravel-notification-channels/discord
```

config/services.php
```
    'discord' => [
        'token'   => env('DISCORD_BOT_TOKEN'),
        'channel' => env('DISCORD_CHANNEL'),
    ],
```

.env
```
DISCORD_BOT_TOKEN=
DISCORD_CHANNEL=
```
チャンネルのIDは開発者モードを有効化するとコピーできるようになる。  
https://support.discordapp.com/hc/ja/articles/206346498-%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC-%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC-%E3%83%A1%E3%83%83%E3%82%BB%E3%83%BC%E3%82%B8ID%E3%81%AF%E3%81%A9%E3%81%93%E3%81%A7%E8%A6%8B%E3%81%A4%E3%81%91%E3%82%89%E3%82%8C%E3%82%8B-

```
php artisan discord:setup
```

```
Is the bot already added to your server? (yes/no) [no]:
> no

What is your Discord app client ID?:
> ...

Add the bot to your server by visiting this link: https://discordapp.com/oauth2/authorize?&client_id=...&scope=bot&permissions=0
```

URLをブラウザで開いてBotをサーバーに追加。最後の`permissions=0`でBotの権限が決まるので今後APIでエラーになる時はここを変更してセットアップし直す。

```
Continue? (yes/no) [yes]:
> yes

Attempting to identify the bot with Discord's websocket gateway...
Connecting to 'wss://gateway.discord.gg'...
Your bot has been identified by Discord and can now send API requests!
```

ここまでで送信はできるのでテスト。artisanコマンドからNotificationを呼ぶ簡単なテスト。

TestNotification
```php
public function toDiscord($notifiable)
{
    return DiscordMessage::create("test");
}
```

TestCommand
```php
public function handle()
{
    \Notification::route('discord', config('services.discord.channel'))
                ->notify(new TestNotification());
}
```

`php artisan discord:test`で送信できてたら成功。

## RestCord
次はRestCordの準備。Laravel用のラッパーもあるけど中身見た上で使わない。

```
composer require restcord/restcord
```

必要なのはBotのtokenだけなのでNotificationsと同じtokenを使う。

毎回`new DiscordClient(['token' => config('services.discord.token')])`とはしたくないのでServiceProviderを作る。
```
php artisan make:provider RestCordServiceProvider
```

```php
public function register()
{
    $this->app->singleton(DiscordClient::class, function ($app) {
        return new DiscordClient([
            'token'  => config('services.discord.token'),
            'logger' => $app['log']->channel()->getLogger(),
        ]);
    });
}
```

これでDIか`app(DiscordClient::class)`でtokenセット済のDiscordClientが得られる。Facadeは使ってない。

RestCordのテスト。
GuildCommand。APIでのGuild=サーバー。Guildの中に複数のチャンネルがある。
```php
public function handle(DiscordClient $client)
{
    $guild = $client->guild->getGuild([
        'guild.id' => config('services.discord.guild'),
    ]);

    dump($guild);

    $channels = $client->guild->getGuildChannels([
        'guild.id' => config('services.discord.guild'),
    ]);

    dump($channels);

    $members = $client->guild->listGuildMembers([
        'guild.id' => config('services.discord.guild'),
        'limit'    => 5,
    ]);

    dump($members);
}
```

ここまで問題なければ通常必要なAPIはすべて使える。

RestCordでのメッセージ送信。
PostCommand
```php
public function handle(DiscordClient $client)
{
    $client->channel->createMessage([
        'channel.id' => (int)config('services.discord.channel'),
        'content'    => 'RestCord test',
    ]);
}
```

## 役職(Role)
想定してるAPIの使い方としてプライベートチャンネルに対してユーザーが参加できるかどうかを管理したい。Discordの場合役職で決まるので役職の付け外しを自動化できればいい。

RoleCommand
```php
public function handle(DiscordClient $client)
{
    //Roleリスト。RoleのIDはここから調べるしかないかも。
    $roles = $client->guild->getGuildRoles([
        'guild.id' => config('services.discord.guild'),
    ]);

    dump($roles);

    //Role追加
    $client->guild->addGuildMemberRole([
        'guild.id' => config('services.discord.guild'),
        'user.id'  => config('services.discord.bot'),
        'role.id'  => config('services.discord.role'),
    ]);

    //プライベートチャンネルへ投稿
    $client->channel->createMessage([
        'channel.id' => (int)config('services.discord.private'),
        'content'    => 'private test',
    ]);

    //Role削除
    $client->guild->removeGuildMemberRole([
        'guild.id' => config('services.discord.guild'),
        'user.id'  => config('services.discord.bot'),
        'role.id'  => config('services.discord.role'),
    ]);

    //削除後プライベートチャンネルへの投稿は失敗
}
```

役職を変更できるかどうかはBotのpermissionsや役職の順番の影響を受けるのでエラーになる時はその辺りを見直す。上位の役職を持ったユーザーしか下位の役職を変更できない。
```
There was an error executing the addGuildMemberRole command: Client error: `PUT https://discordapp.com/api/v6/guilds/.../members/.../roles/...` resulted in a `403 FORBIDDEN` response:
```

## Yasmin
Yasminでメンションへの返信。これもartisanコマンドで作っておく。  
https://github.com/kawax/discord-project/blob/master/app/Console/Commands/ServeCommand.php

ローカルで動かす時は`php artisan discord:serve`で起動、終了はCtrl+C。サーバー上で動かす時はSupervisorでデーモン化。Forgeならデプロイ時に`sudo supervisorctl restart all`でいいはず。
エラーになる時はパスワードなしで実行できるように設定。
```
echo "forge ALL=NOPASSWD: /usr/bin/supervisorctl restart all" > /etc/sudoers.d/supervisorctl
```

リアルタイムなやり取りもできるようになったので普通にBotと言えるレベルのことが可能に。DiscordにはOutgoing WebHookがないのでこの方法しかなさそう。

## Socialite
ついでにSocialiteも作っておいた。  
https://github.com/kawax/socialite-discord

これとほとんど同じだけどイベントを使うSocialiteProvidersはやはり好きになれない。  
https://github.com/SocialiteProviders/Discord
