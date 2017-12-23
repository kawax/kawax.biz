---
title: "React Native を試した"
date: 2017-12-24T03:01:42+09:00
categories: [""]
draft: false
---

https://facebook.github.io/react-native/

## 前提
昔MacアプリとついでのWindowsアプリ作ってたので「クロスプラットフォームは幻想派」。
素人が作れるような簡単なアプリならいいけど複雑になると色々問題が発生する。
なんとかできるような大きな会社以外はOSのアップデートについて行けずどこかで破綻する。
React Nativeもすでにそんな感じで個人がちょっと作ってみたレベルと有名企業のアプリしか見かけない。

- XcodeやAndroid開発環境は一応揃えてあるMac。10.13.2
- node.js 8.9.1
- Reactは軽く

## Getting Started
ドキュメント通りにやっていく。  
ただ、前にReact触った時に見た日本語版の公式ドキュメントが数ヶ月後には消えてたりしたのでReactのドキュメントは信用できない。  
https://facebook.github.io/react-native/docs/getting-started.html


```
npm install -g create-react-native-app
```

```
create-react-native-app react-native-tutorial
cd react-native-tutorial
```

ここですでにドキュメントと変わってる。yarn使ってるので

```
yarn start
```

初回はエラー。

```
See https://git.io/v5vcn for more information, either install watchman or run the following snippet:
  sudo sysctl -w kern.maxfiles=5242880
  sudo sysctl -w kern.maxfilesperproc=524288
```

コマンドを実行するか watchman をインストールする

```
brew update
brew install watchman
```

再度起動
```
yarn start
```

起動後 `i` でiOSシミュレーターで起動。

```
 › Press a to open Android device or emulator, or i to open iOS emulator.
 › Press q to display QR code.
 › Press r to restart packager, or R to restart packager and clear cache.
 › Press d to toggle development mode. (current mode: development)
```


## eject
Getting Started には今回やった Create React Native App を使う方法と react-native-cli を使う方法がある。  
`Building Projects with Native Code` のほう。  
https://facebook.github.io/react-native/docs/getting-started.html

Create React Native App のほうの npm script には `eject` がある。
あくまで最初のプロジェクト作成から開発初期を楽にするツールであるCreate React Native Appで作ったプロジェクトでは色々制限がある。
このままでは配布するアプリにできない。
`eject` でプロジェクトを react-native-cli を使う形に変換できる。一度実行すると手動で書き換えない限り元には戻せない。

アプリにするには `eject` 後にXcodeやAndroid Studioで。この先は React Native 特有なことは少ないので情報はいくらでもある。

## 終わり
具体的に何か作りたいものがあるわけでもないので超基本だけで終わり。
最初と最後のアプリへの出力さえ分かっていれば後は React Native 標準コンポーネントの組み合わせで作るだけなので難しいことはない。ネイティブコードで拡張…？それがまさに地獄の入り口だよ。それができるなら最初からネイティブで作ればいい。

React Native は現時点でv0.51。いわゆる `production ready` とはまだ言えないのでちょっと試しただけの個人や大きな会社以外で React Native でアプリ作ろうとしてる会社は後で大変そう。

使うとしても1.0から。
