---
title: "BonjourでVagrant/Homesteadのhosts設定を不要にする"
date: 2018-07-02T23:25:27+09:00
categories: ["Laravel"]
draft: false
---

数年ぶりにBonjourの文字を見かけてもしかしてVagrantで使えるのでは…と試したらできた。  
最後に見たのはたぶんVagrant誕生よりも前なので完全に記憶から消えてた。  
https://ja.wikipedia.org/wiki/Bonjour

## 環境
- Mac
- Windowsではこの辺りをインストール https://support.apple.com/kb/DL999?locale=ja_JP&viewlocale=ja_JP
- Vagrant / Homestead
- Vagrant boxのOS : Ubuntu 18.04

## Avahi
https://ja.wikipedia.org/wiki/Avahi

やることはVagrant box内でavahiをインストールするだけ。

```
sudo apt install avahi-utils
```

avahi-browseでMac側が出てくる。

```
avahi-browse -a -t
```

avahiの設定とかはなにもない。

## Homestead
Homestead.yamlの`hostname`に`.local`を加えたドメイン（例えばhomestead.local）ですでに表示可能。  

httpsにできるかは未確認。

hostsの設定なし。  
Laravelプロジェクト作りすぎてhostsが何十個もあるのでちょうどいいかもしれない。  
「hostsの設定」と「avahiインストール」どっちの手間を取るかでしかないけど。  
OAuthのリダイレクトで`.local`が使えないようなこともまれにあるので結局は使い分けることになる。
