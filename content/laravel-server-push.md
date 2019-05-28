---
title: "Laravel Server Push Middleware"
date: 2019-05-28T20:51:29+09:00
categories: ["Laravel"]
draft: false
---

他人のcomposerパッケージを少し修正して使いたい場合、GitHubでフォークしてcomposer.jsonで設定すれば使える。
https://getcomposer.org/doc/05-repositories.md#vcs

一時的ならこれでいいけど長く使って使用プロジェクトも増えた場合は段々面倒になる。
update時の動作見てるといちいちGitHubまで見に行ってるような…。

オリジナルに戻せないほど使ってたらもう別パッケージとして自分の管理下に置くのがいい。

最近作ってたLaravel Server Push Middlewareもそんな感じで。
中身は作り直してるけど。

- 流石にもうElixirがデフォルトはないのでLaravel Mixがデフォ。
- configファイルはそのまま使えるようにしたけどMixで使うだけならconfigファイルの公開は不要。

https://github.com/kawax/laravel-server-push