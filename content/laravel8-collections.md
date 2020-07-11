---
title: "Laravel8でバージョンアップが面倒な変更が入りそうになったけど回避された"
date: 2020-07-11T19:20:03+09:00
categories: ["Laravel"]
draft: false
---

きっかけは不明なもののCollectionを`illuminate/collections`に分離したかったんだろう。
https://github.com/laravel/framework/tree/addaecdbd10ed774e0a37f25f0c7be52fbcd2483/src/Illuminate/Collections
`Arr`も必要なので移動。
`Macroable`も必要だけどこれは他でも使ってるので専用に`illuminate/macroable`に分離。

名前空間も変わって
`Illuminate\Collections\Collection`
`Illuminate\Collections\Arr`
`Illuminate\Macroable\Macroable`
元も非推奨で残ってるけどいずれ削除される。

普通にLaravelを使ってるならCollectionは`collect()`から使う、Macroableは使わないので影響があるとしたらArr。
ヘルパー削除で大量に修正させた後にまた修正させる気か？と思ってたけどさすがにこれはないよなってことで戻された。
https://github.com/laravel/framework/pull/33108
`illuminate/collections`と`illuminate/macroable`の分離は変わらず名前空間だけ戻された。
大きな修正は不要なはず。

## パッケージ開発者向け
`"illuminate/support": "*"`での指定がしにくくなった。戻されたので8では影響ないけど今後も急に変わる可能性はある。
`"illuminate/support": "^6.0||^7.0||^8.0"`で指定していくしかないな。