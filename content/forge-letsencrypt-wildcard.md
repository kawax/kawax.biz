---
title: "Laravel Forge が Let's Encrypt ワイルドカードドメインに対応していた"
date: 2018-03-21T18:11:23+09:00
categories: ["Forge"]
draft: false
---

<img src="/img/forge-letsencrypt-wildcard/forge-letsencrypt-wildcard1.png">

ただしワイルドカードのためにはDNS検証が必要でRoute53など一部にしか対応してないらしい。`*.`を消したらDNS Provider用の設定が消えた。

<img src="/img/forge-letsencrypt-wildcard/forge-letsencrypt-wildcard2.png">

今まで通りに。

ということで使うならRoute53も必須。個人用はあまりRoute53使ってないので必要になったら考える。

[Wildcard LetsEncrypt Certificates on Forge](https://medium.com/@taylorotwell/wildcard-letsencrypt-certificates-on-forge-d3bdec43692a)
