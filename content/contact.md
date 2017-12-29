---
title: "連絡手段"
date: 2017-07-22T14:51:40+09:00
categories: ["Contact"]
draft: false
---

基本的にはチャットのみ。完全リモートで当たり前の時代。

<!--more-->

- ChatWork https://www.chatwork.com/kawax
- Slack
- kawaxbiz@gmail.com

電話や Skype は何があろうと絶対に使わない。もはやスマホの SIM に通話機能自体ない。

### 条件
https://scouty.co.jp/ みたいな所からスカウトメールが来るので一応書いておくと「一度も会わないこと」が条件。面接からなんて無駄な時間を使う気はないので具体的なプロジェクトがあって依頼したい場合のみメールしてください。

## Netlify のフォーム
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
