---
title: "Hugo + Vue.js + Laravel Mix"
date: 2017-09-06T22:53:00+09:00
categories: ["Hugo", "Vue.js", "Laravel"]
draft: false
---

<!--more-->

## 目的
Hugo サイト内で少しだけ動的な機能が欲しい。

## デモ
https://hugo-vue-mix.netlify.com/

## GitHub
https://github.com/kawax/hugo-vue-mix

大体 README に書いたのでここに書くことはあまりない。

- Laravel 外でも Mix 使うと楽。
- Netlify は GitHub へ push だけでデプロイ。独自ドメイン設定できるし当たり前に https 対応。
- 現在 WordPress で作ってるようなサイトを置き換えられるように作ったけど問題は WordPress しか使えないような人では使えない。
- すべて静的なファイルなので不正ログインも改ざんも不可能。WordPress 捨てるだけで大量のメリットが。
