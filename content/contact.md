---
title: "連絡手段"
date: 2018-01-29T14:51:40+09:00
categories: ["Contact"]
draft: false
---

SlackやDiscordのテキストチャットのみ。

法にしか従わないのでフリーランスではリモートが絶対条件。労働契約なら別。

## スキル
LAPRASがちょうど良かったので貼る。
[タグとスコアについて](https://talent-help.lapras.com/lapras-%E3%83%98%E3%83%AB%E3%83%97/%E7%94%BB%E9%9D%A2%E3%81%A8%E3%82%B9%E3%82%B3%E3%82%A2%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/%E3%82%BF%E3%82%B0%E3%81%A8%E3%82%B9%E3%82%B3%E3%82%A2%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)
![LAPRAS](/img/lapras3.png)

イベントには一切参加しないので評価なしなのが正しく反映されてる。

Findyのスキル偏差値は75に張り付いたまま変動しない。

自分から登録しないのでこういうGitHubから勝手に分析するサービスでしか客観的な数字がない。



色々なサービスにバラバラなIDで登録してるのでkawax名義での活動記録はGitHubに登録した2012年以降くらいしかない。今でもメインのIDは別。LAPRASでもメインIDは見つけられてないので繋がってない。

- Twitter https://twitter.com/kawaxbiz
- https://github.com/kawax
- https://qiita.com/kawax
- https://teratail.com/users/kawax

<!--
## 最近の業務内容
- Laravel(2013~)
- Vue.js
- AWS(2012~)
- WordPressの運用
- Golang（少し）

## 昔やってたこと
- C。大学で習っただけなので当時は理解はしてなかったけど基礎として役に立ってる。
- Perl。CGI時代。htmlもこの辺だけど「日本語ができる」と同レベルなので書くまでもない。
- PHP3~4~5初期。素のPHP時代。この頃に仕事で使ってなくて良かった。
- クロスプラットフォームツールでのMac/Windowsアプリ開発。まだ残ってるけど頻繁に名前が変わって今は読み方もよく分からないものに。
- 現Xcode。当時はProject Builder/Interface Builder。Objective-Cの頃。
- Movable Type。ブログ誕生の頃。今考えるとPerl+PHPのシステム。
- アプリを公開してそのサイトをPHPで作ってた時代。その後OSの移行やスマホ普及もあり特定のOSでしか動かないアプリよりwebのほうがいいとなっていく。
- Zend Framework ver1。他PHPのフレームワーク。昔Zendで作ったものは全部Laravelで作り直した。
- 過去の遺産は気にする必要ないので以降はLaravelに全力。
- AngularJS ver1。node.js登場後のフロント周りはgrunt、CoffeeScript時代から色々使ってきて今はなるべくシンプルな方向に。道具自体に詳しくなっても意味はない。
- Ionicでブラウザゲームとスマホアプリ化。当時のスマホスペックでは厳しかったので途中で断念。
- スマホアプリのサーバーサイド。スマホアプリに行かなかったのは個人で気軽に公開できないからだけどスマホでも結局サーバーサイドが必要だった。
- Railsの運用。Amazon ECS。

ほとんどは一人から数人のチームでやってたこと。企業の手伝いが増えたのはAWSやGitHubの登場後。リモートで何も困らなくなったから。

関係あることだけ書いてるけど実際は他にも色々。

-->


## フォーム

<form name="contact" netlify>
  <div class="field">
    <label class="label">名前</label>
    <div class="control">
      <input name="name" class="input" type="text" required>
    </div>
  </div>

  <div class="field">
    <label class="label">メール</label>
    <div class="control">
      <input name="email" class="input" type="email" required>
    </div>
  </div>

  <div class="field">
    <label class="label">メッセージ</label>
    <div class="control">
      <textarea name="message" class="textarea" required></textarea>
    </div>
  </div>

  <div class="field">
    <div class="control">
        <button class="button is-primary">送信</button>
    </div>
  </div>

</form>
