---
title: "連絡手段"
date: 2018-01-29T14:51:40+09:00
categories: ["Contact"]
draft: false
---

SlackやDiscordのテキストチャットしか使わない。

<!--

法にしか従わないのでフリーランスではリモートが絶対条件。労働契約なら別。

## スキル
LAPRASがちょうど良かったので貼る。
[タグとスコアについて](https://talent-help.lapras.com/lapras-%E3%83%98%E3%83%AB%E3%83%97/%E7%94%BB%E9%9D%A2%E3%81%A8%E3%82%B9%E3%82%B3%E3%82%A2%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/%E3%82%BF%E3%82%B0%E3%81%A8%E3%82%B9%E3%82%B3%E3%82%A2%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)
![LAPRAS](/img/lapras3.jpg)

イベントには一切参加しないので評価なしなのが正しく反映されてる。

自分から登録しないのでこういうGitHubから勝手に分析するサービスでしか客観的な数字がない。



色々なサービスにバラバラなIDで登録してるのでkawax名義での活動記録はGitHubに登録した2012年以降くらいしかない。今でもメインのIDは別。LAPRASでもメインIDは見つけられてないので繋がってない。

- Twitter https://twitter.com/kawaxbiz
- https://github.com/kawax
- https://qiita.com/kawax
- https://teratail.com/users/kawax


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
