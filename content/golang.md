---
title: "Golang を始める"
date: 2017-12-27T19:41:59+09:00
categories: ["golang"]
draft: false
---

http://golang-jp.org/

## 理由
webはもうほとんどLaravelでいいけど足りない所を補完できる言語として `Go` が良さそうと気付いた。

- あまりないけどPHPでは足りない速度が欲しい時。
- コンパイル後にバイナリファイル一つにできるのでLaravelから exec で実行してもいい。
- 別のAPIとして作って呼び出してもいい。AWS LambdaがGo対応予定。
- CLIツールを作りたい時。

意外とバイナリ一つなのが大きくて。Rubyやnode.jsだとサーバー構築もしっかりしておかないと動かせない。GoはたぶんLaravelプロジェクトにそのまま入れるだけでもいい。

## インストール
公式サイトからダウンロードもできるけど brew でできたので brew で。

```
brew install go
```

```
==> Pouring go-1.9.2.high_sierra.bottle.tar.gz
==> Caveats
A valid GOPATH is required to use the `go get` command.
If $GOPATH is not specified, $HOME/go will be used by default:
  https://golang.org/doc/code.html#GOPATH

You may wish to add the GOROOT-based install location to your PATH:
  export PATH=$PATH:/usr/local/opt/go/libexec/bin
==> Summary
🍺  /usr/local/Cellar/go/1.9.2: 7,646 files, 293.9MB
```

ドキュメント見てPATHの設定。  
http://golang-jp.org/doc/code.html

ホームのSitesにした。

```
export PATH=$PATH:/usr/local/opt/go/libexec/bin
export GOPATH=$HOME/Sites/go
export PATH=$PATH:$GOPATH/bin
```

## パッケージパス
もはや当たり前にGitHub。

```
mkdir -p $GOPATH/src/github.com/kawax
```

## hello

```
mkdir $GOPATH/src/github.com/kawax/go-hello
cd ~/Sites/go/src/github.com/kawax/go-hello
```

hello.go を作って

```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```

```
go install
```

$GOPATH/bin に生成されている。

PATHを設定してるのでこれだけでも実行可能。

```
go-hello
```

このファイルがもう単体で使える。といってもコンパイルしたOS用なので別OS用は `GOOS` や `GOARCH` を指定する。

```
GOOS=linux GOARCH=amd64 go build
```

ここまでできればLaravelから使う用は作れるのでひとまず終わり。
