---
title: "Laravel5.8へアップグレード"
date: 2019-02-27T13:53:36+09:00
categories: ["Laravel"]
draft: false
---

5.8は事前に準備しておけばすぐ終わる。

## いつもの見る所
- https://laravel.com/docs/5.8/releases
- https://laravel.com/docs/5.8/upgrade
- https://readouble.com/laravel/5.8/ja/releases.html
- https://readouble.com/laravel/5.8/ja/upgrade.html
- https://github.com/laravel/laravel/compare/5.7...master

## キャッシュ時間
5.8で一番重要なのはここ。現在5.5-5.7使っててすぐに5.8にしないつもりでもここだけは今から準備してたほうがいい。

```php
// Laravel 5.7 - Store item for 30 minutes...
Cache::put('foo', 'bar', 30);

// Laravel 5.8 - Store item for 30 seconds...
Cache::put('foo', 'bar', 30);

// Laravel 5.7 / 5.8 - Store item for 30 seconds...
Cache::put('foo', 'bar', now()->addSeconds(30));
```

5.7までは「分」。5.8からは「秒」。修正しなくてもエラーにはならないけどキャッシュ時間が短くなるという後からでは見つけにくい変更。後から他人が見たら「分」と「秒」どっちのつもりで書いたのか分からない。

対応としては`now()->addMinutes(30)`のようなDateTimeでの指定にする。

5.8に上げる前に修正したほうがいい。

## アトミックロック
これもキャッシュ関連。使ってる所はあまり見たことないけど使ってるなら修正。

キャッシュの普通の使い方でもロックはできる。Cache::lock()がキャッシュの機能なのはその辺りからだろう。

## 複数単語のモデル
Mediumとは言ってるけど絶対影響大きい…。

```php
// Laravel 5.7
App\Feedback.php -> feedback (correctly pluralized)
App\UserFeedback.php -> user_feedbacks (incorrectly pluralized)

// Laravel 5.8
App\Feedback.php -> feedback (correctly pluralized)
App\UserFeedback.php -> user_feedback (correctly pluralized)
```

モデル側で$tableを指定。テーブル名自体を変更してもいいけど。

```php
/**
 * The table associated with the model.
 *
 * @var string
 */
protected $table = 'user_feedbacks';
```

## ヘルパー
修正するのは大変なのですでに大量に使ってるならそのまま使えばいい。
修正したつもりでも修正漏れが残る。
https://kawax.biz/laravel58-helpers/

パッケージ開発者はヘルパー使わないように修正。

## Deferred Service Providers
これもパッケージ開発者向けだけど5.8以上にしか対応できないので今後メジャーバージョンを上げるパッケージが増えそう。
`composer update`だけでは気付かないので`composer outdated`で確認。

## Markdown File Directory Change
公開して使ってるなら。

## Nexmo / Slack Notification Channels
使ってるなら。
Slackは使ってる人多そう。2.0になってるので事前に追加してた人は修正が必要。

```
composer require laravel/slack-notification-channel
```

## プロジェクト側
比較ツール見て必要な箇所だけ変更すればいい。

- migrationsがbigIncrements。既存プロジェクトには関係ないけど。毎回自分でbigIncrementsに変えてた。
- パスワードが8文字以上
- public/svg/が削除されてまたエラー画面が変わった

## その他
いつものようにこれ以上の細かい所はそれぞれのプロジェクトで使ってる機能次第。

1プロジェクトなら早いけどもう数が多すぎる…。
どうせサードパーティパッケージの対応待ちになるので徐々に上げていく。
