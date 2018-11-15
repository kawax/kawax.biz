---
title: "Symfony 1.0 で作られたシステムを Laravel 5.4 で作り直した話"
date: 2018-10-09T02:34:21+09:00
categories: ["Laravel"]
draft: true
---
（Qiita/Kobitoからサルベージ。元は2017/01）

中身の具体的なことは書けないけど。

## 元のシステム
- PHP 5.3.3 (2010年リリース／2014年サポート終了)
- Symfony 1.0.20 (2009年リリース／2010年サポート終了)
- どこかのVPS

2013年に作ったシステムでSymfony 1.0なのは本当に意味が分からない…。
一般公開せずBasic認証で制限してるので2017年まで使ってたけど、そろそろ動かない所も出てきたので修正して欲しいと言われてもこんなのをどうにかするのは無理なので全部作り直し。

ドキュメントなし、gitなんて使ってない、実際に動いてるシステムがあるだけ。

## DBをコピー
既存DBからLaravelのmigration fileを作るSequel Pro用のプラグインみたいなもの
https://github.com/cviebrock/sequel-pro-laravel-export

zipでダウンロードしてダブルクリックでインストールされる。
使い方はこの記事で十分。
http://co.bsnws.net/article/203

各テーブルごとにEloquentのモデルを作る。
$tableで指定すればLaravelのルールと違うテーブル名でも使える。

```
    protected $table = 'テーブル名';
```

幸い主キーは全部 `id` だったのでそのまま。

元のDBからエクスポート→ローカルのDBにインポート。
Eloquentから使える所まで確認。

## PHP5.3
phpスクリプトをcronで回してる所があるのでこれをローカルで動かすためにanyenv+phpenvでPHP5.3をインストール。

このphpスクリプトはログインしてCSVをダウンロードしてるけど
PEARのHTTP_Request使ってCSVはexplodeでぐるぐる回してて本当に勘弁して欲しい…。
ファイルの読み書きはfopen fcloseって一体いつの時代…。
これSymfony全く関係ない…。

ここは後で作り直すので今動いてない部分だけ修正して終わり…だけど結局かなり書き換えた。
PHP5.3で動くGoutte1.0をpharで、とCSVはSplFileObject。
composerは使えるけどここでそこまではしなくていいか。

あ、5.3なのでarrayで[]が使えない…。

## DB再設計
元のDBのままでは使いにくいので作り直し。
複数のテーブルに分かれてるのを一つのテーブルに…と作りかけて気付く。
元のCSVを再ダウンロードしたほうがいいのでは…。
CSVの仕様変更によりDBのデータは結構壊れてるし。

過去のCSVも全部DLできることを確認したので元のシステムからは何も引き継がず全部捨てることに決まった…。

## 仕切り直し
一から作るなら簡単。
すでにSymfonyからって話ではなくなってるので大まかな方針だけ。

今回はcronで動かす部分が多いので
まずartisanコマンドから作る。
artisanコマンドからキュージョブをsyncで同期的に呼び出す。
何も返さない場合小さいジョブ単位で分離できるから分かりやすいかも。
後から非同期にもできる。

```php
$job = (new MyJob())->onConnection('sync');

dispatch($job);
```

何か返すControllerから呼び出すような機能はサービス層にでも作る。
ジョブ内でこれはできない。

```
return response()->json($data);
```


`fabpot/goutte`最新バージョンと`league/csv`を遠慮なく使える。

後はタスクスケジュールで定期的に動かせばデータ取得部分は終わり。
タスクスケジュールは便利すぎてLaravelの好きな所ベスト3には入る。

データ表示はAPIとしてjson返すだけ。
（Laravel Passport使ってみたけどこれも便利…。）
それをVue.jsで表示。
vuetable-2
https://github.com/ratiw/vuetable-2
Laravelでのサンプルがあるのでこれと同じようにすればいいはず。
https://github.com/ratiw/vuetable-2-with-laravel-5.4
もしくはこれか。
http://element.eleme.io/#/en-US

…Elementで進めてみたけどかなりいいな。
今回はこれで行く。

## まとめと追加ネタ
2週間くらいで元と同じ所まで作り直せた。
Symfonyの部分は全く見てない。
どうしようもないものは作り直したほうが早い。

とはいえDB捨てられない場合も多いだろうから既存DBからの移行の参考になるかもしれないので書いたものは残しておく。

Seederでこんな風にするとエクスポートしたSQLをインポートできる。
migrationしたテーブルを上書きして消すことになるけど。

```php
DB::unprepared(file_get_contents(__DIR__ . "/export.sql"));
```


どういうやり方にせよ `php artisan migrate:refresh --seed` で何もないDBから元のDBを再現できるようにしておくと便利。
元のシステム動かしつつ新システム作る。
新システムへの移行時に元DBの最終データをこれでインポート。
DB変換をartisanコマンドで作っておけば移行しやすいはず。
