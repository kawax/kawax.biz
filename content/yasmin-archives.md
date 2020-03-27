---
title: "PHP用DiscordライブラリYasminが消えていたのでフォーク版を公開した"
date: 2020-03-27T20:42:42+09:00
categories: ["Discord"]
draft: false
---

数ヶ月前にpackagistは破棄されてGitHubはArchivedだった。
念の為フォークしておいてよかった…。

https://github.com/laravel-discord
PHP用だけどLaravelから使うことしか想定してないのでlaravel-discord。
これから使ってる。
https://github.com/kawax/discord-manager

単純なメッセージ送信だけならYasmin使う必要はないけどメッセージを受け取って返信するようなbotはPHPではYasminしかできないはず。
他のチャットなら簡単だけどDiscordはWebSocketなので色々と厄介。
そういう貴重なパッケージをいきなり消すのはやめて欲しい。