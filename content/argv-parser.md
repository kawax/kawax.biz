---
title: "コマンドライン引数を解析する"
date: 2019-01-26T19:45:56+09:00
categories: ["Laravel"]
draft: false
---

Laravelのartisanコマンドでは

こう定義して
```
protected $signature = 'email:send {user} {--queue}';
```
こう実行すると
```
php artisan email:send 1 --queue
```
コマンド内で取得できる。
```
$userId = $this->argument('user');
$queueName = $this->option('queue');
```

https://laravel.com/docs/master/artisan
https://readouble.com/laravel/5.7/ja/artisan.html

定義と入力から値を取得する部分をコマンド以外の任意の場所でできるようにする。
webでユーザーの入力を解析したりチャットボットのコマンドで使える。

## 使うもの
Laravelと同じ仕様なのでilluminate/consoleをそのまま使う。
https://github.com/illuminate/console
さらにベースのSymfonyのドキュメントも見ておいたほうがいい。
https://symfony.com/doc/current/components/console.html
https://github.com/symfony/console

Laravelなら何もインストールしなくていい。
Laravel以外ならcomposerで`illuminate/console`をインストール。

## コード
いきなり最終的なコード。

```php
use Illuminate\Console\Parser;

use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputDefinition;
use Symfony\Component\Console\Input\ArgvInput;

$signature = 'email:send {user} {--queue}';
$command = 'email:send 1 --queue';

$argv = explode(' ', $command);

[$name, $args, $options] = Parser::parse($signature);

$definition = new InputDefinition();
$definition->setArguments($args);
$definition->setOptions($options);

$input = new ArgvInput($argv, $definition);

$userId = $input->getArgument('user');
$queueName = $input->getOption('queue');
```

Laravelと同じようにすればいい。
https://github.com/illuminate/console/blob/8ffd2310a60472ed060594824fd231924a8a2c2c/Command.php#L124

explodeで単純に分割してるだけなのでスペースを含む場合はもう一工夫必要かもしれない。
Symfonyは`$_SERVER['argv']`で取得してる部分なので。
https://github.com/symfony/console/blob/c195108dddab91f51b9c57f63e8b184d0a257a54/Input/ArgvInput.php#L50

注意点としては$argvの最初にコマンド名が来るようにする。入力の時点で`<user> /cmd test`のような関係ない部分が含まれてる場合は事前に取り除く。

## 実際の使用例
Discord botのコマンドで使ってる。traitにして必要なコマンドでのみ有効化。
https://github.com/kawax/discord-manager/blob/master/src/Concerns/Input.php

コマンド側では$argvまで用意。

```php
$argv = explode(' ', str_after($message->content, config('services.discord.prefix')));
$input = $this->input($argv);

$userId = $input->getArgument('user');
$queueName = $input->getOption('queue');
```
