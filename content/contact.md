---
title: "連絡手段"
date: 2017-07-22T14:51:40+09:00
categories: ["Contact"]
draft: false
---

基本的にはチャットのみ。フルリモートで当たり前の時代。

- ChatWork https://www.chatwork.com/kawax
- Slack
- kawaxbiz@gmail.com

電話や Skype は何があろうと絶対に使わない。もはやスマホの SIM に通話機能自体ない。

## 今やりたいこと
- Laravel
- AWSを全力で使える規模のシステムの運用
- Vue.js
- Golang

## 今やってること
- Laravel + Vue.js での開発
- AWS
- 大量のWordPressの運用
- Railsアプリの運用

## 昔やってたこと
- C
- Perl
- クロスプラットフォームツールでのMac/Windowsアプリ開発。まだ残ってるけど頻繁に名前が変わって今は読み方もよく分からないものに。
- 現Xcode。当時はProject Builder/Interface Builder。
- Zend Framework
- AngularJS ver1
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
