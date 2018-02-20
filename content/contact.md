---
title: "連絡手段"
date: 2018-01-29T14:51:40+09:00
categories: ["Contact"]
draft: false
---

基本的にはチャットのみ。フルリモートで当たり前の時代。

- ChatWork https://www.chatwork.com/kawax
- Slack
- kawaxbiz@gmail.com

電話や Skype は何があろうと絶対に使わない。もはやスマホの SIM に通話機能自体ない。
リモートOKと言いつつ最初に面談を要求する会社の相手はしない。今まで関わった会社はどことも一度も会ったことない。

このくらい書いておかないと理解してない問い合わせが来る。

## 今やりたいこと(2018)
- Laravel
- AWSを全力で使える規模のシステムの運用
- Vue.js
- Golang

## 今やってること
- エンジニアのいない小さな会社の専属エンジニアのような立場。リモートで完結するなら安くてもいいという条件で複数掛け持ち。
- Laravel + Vue.js 一人で開発。
- AWS。一人で管理してるので全部自力で調査。誰かに言われた通りのことをやってるだけではない。
- 大量のWordPressの運用
- Railsアプリの運用

## 昔やってたこと
- C
- Perl。CGI時代。
- PHP3~4~5初期。素のPHP時代。
- クロスプラットフォームツールでのMac/Windowsアプリ開発。まだ残ってるけど頻繁に名前が変わって今は読み方もよく分からないものに。
- 現Xcode。当時はProject Builder/Interface Builder。Objective-Cの頃。
- Movable Type。ブログ誕生の頃。今考えるとPerl+PHPのシステム。
- Zend Framework ver1。他PHPのフレームワーク。
- AngularJS ver1。node.js登場後のフロント周りはgrunt、CoffeeScript時代から色々使って今はなるべくシンプルな方向に。
- スマホアプリのサーバーサイド

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
