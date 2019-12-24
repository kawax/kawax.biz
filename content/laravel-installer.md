---
title: "laravel/installer がメジャーバージョンアップした時の対応"
date: 2019-12-24T16:57:32+09:00
categories: ["Laravel"]
draft: false
---

2019年5月にv2.0、11月にv3.0が出てる。プロジェクトのcomposerは更新してもglobalのほうは気にしてない人が多い。v1.xがインストールされたままな人も多いと思う。

最新バージョンがあるか確認

```
composer global outdated
```

globalのcomposer.jsonを直接書き換えるか再度インストールすれば最新バージョンになる。

```
composer global require laravel/installer
```

全体を更新。

```
composer global update
```

## 気になった変更点
`--auth`付けると最初から`laravel/ui`も加えた状態でプロジェクトが作られる。

```
laravel new project --auth
```

そもそも`laravel/laravel`のrequire-devに加えて欲しかったけどinstallerのフラグで対応された。
Laravel5.8までと同じように作るなら`--auth`を付ける。
