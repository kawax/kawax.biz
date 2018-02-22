---
title: "Laravel5.6 Logger カスタムチャンネルを作る"
date: 2018-02-22T18:55:48+09:00
categories: ["Laravel"]
draft: false
---

Laravel5.6のLogはManagerベースになったと思ってたけどよく見たら違った。
https://github.com/laravel/framework/blob/5.6/src/Illuminate/Log/LogManager.php

```php
class LogManager implements LoggerInterface
```

確かに継承してない。  
ということは拡張する場合も`extend()`ではない方法。  
ドキュメントに書いてあるのでその通りにするだけ。  
https://readouble.com/laravel/5.6/ja/logging.html

プロジェクト内で使うだけなら`app`下に作って使えばいいけど別プロジェクトでも使えるようにcomposerパッケージにした。

デモプロジェクト  
https://github.com/kawax/laravel-logger-project

CloudWatch Logs  
https://github.com/kawax/laravel-logger-cwlogs  
誰か作るだろうけど。

ChatWork  
https://github.com/kawax/laravel-logger-chatwork  
全部ChatWorkに流すと多すぎだけどログレベルが高い場合のみSlackに流す例が載ってたのでそれならChatWorkもありだろうと判断。ChatWork用のMonolog Handlerがなさそうだったのでそこも適当に作った。

composerでインストールして`config/logging.php`と`.env`で設定するだけ。  
5.6からの`stack`チャンネルで複数同時に流せるのでCloudWatch Logsには全部流しつつ重要なログだけチャットに流すことができる。
