---
title: "連絡手段"
date: 2018-01-29T14:51:40+09:00
categories: ["Contact"]
draft: false
---

- ChatWork https://www.chatwork.com/kawax
- Slack
- kawaxbiz@gmail.com

こうやって公開してるとメールが来るけど「電話」や「常駐」は一発アウトのNGワード。
もうずっとネット上だけで活動してるし会社に頼らず仕事してるのでわざわざ変な相手とは関わらない。

## 今やりたいこと(2018)
- Laravel
- AWSを全力で使える規模のシステムの運用
- Vue.js
- Golang

## 今やってること
- Laravel（2013〜）
- Vue.js（2016〜）
- AWS（2012〜）一人で管理してるので全部自力で調査。誰かに言われた通りのことをやってるだけではない。
- 大量のWordPressの運用
- Railsアプリの運用

2015頃からはLaravel+AWSで開発と運用。今でもバージョンアップを継続中。依頼で作ったシステムは非公開のGitLabに置いてるので公開できない。

## 昔やってたこと
- C。大学で習っただけなので当時は理解はしてなかったけど基礎としてずっと役に立ってる。
- Perl。CGI時代。htmlもこの辺だけど「日本語ができる」と同レベルなので書くまでもない。
- PHP3~4~5初期。素のPHP時代。この頃に仕事で使ってなくて良かった。
- クロスプラットフォームツールでのMac/Windowsアプリ開発。まだ残ってるけど頻繁に名前が変わって今は読み方もよく分からないものに。
- アプリを公開してそのサイトをPHPで作ってた時代。その後Mac OS Xへの変化もあり特定のOSでしか動かないアプリよりwebのほうがいいとなっていく。
- 現Xcode。当時はProject Builder/Interface Builder。Objective-Cの頃。
- Movable Type。ブログ誕生の頃。今考えるとPerl+PHPのシステム。
- Zend Framework ver1。他PHPのフレームワーク。昔Zendで作ったものは全部Laravelで作り直した。
- AngularJS ver1。node.js登場後のフロント周りはgrunt、CoffeeScript時代から色々使って今はなるべくシンプルな方向に。
- スマホアプリのサーバーサイド

色々なサービスにバラバラなIDで登録してるのでkawax名義での活動記録はGitHubに登録した2012年以降くらいしかない。今でもメインのIDは別。

- https://github.com/kawax
- https://qiita.com/kawax
- https://teratail.com/users/kawax

## 使用OS
基本的にはすべて最新バージョンだけどAndroidはNexusが終了してどうなるか分からない。

- Mac
- Windows
- iOS (iPad Pro)
- Android (Nexus)


## フォーム

<form name="contact" netlify>
  <div class="field">
    <label class="label">名前</label>
    <div class="control">
      <input name="name" class="input" type="text" required>
    </div>
  </div>

  <div class="field">
    <label class="label">メール</label>
    <div class="control">
      <input name="email" class="input" type="email" required>
    </div>
  </div>

  <div class="field">
    <label class="label">メッセージ</label>
    <div class="control">
      <textarea name="message" class="textarea" required></textarea>
    </div>
  </div>
  
  <div class="field">
    <div class="control">
        <button class="button is-primary">送信</button>
    </div>
  </div>

</form>
