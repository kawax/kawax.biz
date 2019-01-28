---
title: "cliツールでのhosts設定"
date: 2019-01-28T13:17:06+09:00
categories: [""]
draft: false
---

VagrantならBonjour使えばhosts設定不要になるけどDockerだと難しそうだったのでhostsに戻って設定を楽にできる方法がないか探したら見つけた。

使うもの
https://github.com/cbednarski/hostess

## インストール
Macなら`brew install hostess`でもいい。
他はGitHubからダウンロード。Golang製なのでファイル一つ。

普通の使い方はREADME読めばいいだけなので省略。

```
hostess add local.example.com 127.0.0.1
```

## カレントディレクトリ名で追加
READMEには書いてないようだけど`/etc/hosts`の書き換えなのでsudoでパスワードが必要。

```
sudo hostess add ${PWD##*/}.test 127.0.0.1
```

プロジェクトディレクトリで使う想定。

## makeコマンドから使用
こっちを先に作ったのでMakefileの変数使う方法。

```
hosts:
	sudo hostess add $(notdir $(CURDIR)).test 127.0.0.1
```

`make hosts`

## どうでもいい余談
数年前までプログラミング関連の検索結果はまだ使えると言われてたけど今はもうノイズだらけ。
ただそれも言語の入門やフレームワークの使い方レベルの浅い所だけ。
ちょっと潜るだけでノイズはなくなる。
この記事くらいの全然難しくないことでも邪魔されずに調べられる。


公式情報が一次ソース、公式を元にした情報が二次ソース、それ以下は三次ソースで信用できない。
あまりにも常識中の常識なんだけどエンジニア目指してるような人でさえ分かってない。
自分が何かを答える時は公式を元にして二次ソースになろうとする。
