---
title: "PHP7.4にバージョンアップ後にPhpRedisを再インストール"
date: 2019-11-30T22:26:55+09:00
categories: ["Laravel"]
draft: false
---

Laravel6以降はPhpRedis使うように推奨されているので最近はPhpRedis使ってるけどやはりPHPバージョンアップ時に多少作業が必要だった。

## 環境
- Laravel forge
- Ubuntu 18.04
- PHP7.3から7.4.0にバージョンアップ
- PhpRedis 5.1.1

## 再インストール
PHPをバージョンアップしただけだとredis.soが読み込めてない。

```
$ php -v
PHP Warning:  PHP Startup: Unable to load dynamic library 'redis.so' (tried: /usr/lib/php/20190902/redis.so (/usr/lib/php/20190902/redis.so: cannot open shared object file: No such file or directory), /usr/lib/php/20190902/redis.so.so (/usr/lib/php/20190902/redis.so.so: cannot open shared object file: No such file or directory)) in Unknown on line 0
```

一旦アンインストールしてから

```
pecl uninstall redis
```

再度インストールすれば`/usr/lib/php/20190902/redis.so`にインストールされて使えるようになる。

```
pecl install redis
```

`/etc/php/7.4/`以下のredis.iniファイルはバージョンアップ時に自動でコピーされてるのかも。30-redis.iniにされてるけど。

これだけとはいえ若干面倒。
predis再開するかもしれないのでLaravelでのサポート残して欲しい。
https://github.com/nrk/predis/issues/587