---
title: "WebComponentsの使い所"
date: 2019-03-23T15:05:58+09:00
categories: ["Laravel"]
draft: false
---

[Laravel職人を探す](https://artisans.kawax.biz/)用の外部サイトで使うためのウィジェットをWebComponentsで作ってみた。
https://widget.kawax.biz/
（Laravel Mixは使ってるけどLaravelは関係ない）

WebComponentsなのでJSファイル一つ読み込めばどこでも使える。

```html
<script src="https://widget.kawax.biz/artisans.js" defer></script>
<artisans-user limit="5"></artisans-user>
<artisans-post limit="5"></artisans-post>
```

<script src="https://widget.kawax.biz/artisans.js" defer></script>
<artisans-user limit="5"></artisans-user>
<artisans-post limit="5"></artisans-post>

WebComponentsはこのくらいの使い方がちょうどいい。ReactやVue.jsを捨てて全部置き換えるなんて未来はたぶんない。

サービス内と次に作る予定のChrome拡張はVue.jsを使うし。

## 動作確認済
Mac/Windows/iOSとChrome/Firefox/Safari/Edgeで表示できることは確認した。  
EdgeではまだPolyfillが必要。

Android端末は少し前に壊れたので検証不能。

## CSSの扱い
WebComponents的には`<style></style>`内で書くのが一番良いんだろうけどCSSだけはもう自分で書きたくないのでscssから作ったcssを読み込んでる。

```javascript
import { html } from 'lit-html'

export default () => {
  const url = 'https://widget.kawax.biz/css/artisans.css'

  return html`<link href="${url}" rel="stylesheet">`
}
```

```javascript
import { html } from 'lit-html'

export default () => {
  const url = 'https://widget.kawax.biz/css/artisans.css'

  return html`<style>
  @import "${url}"
  </style>`
}
```

どっちでもいい。これでもコンポーネント内に閉じたCSSは実現できるけど一瞬反映が遅れる。  
CSSを直接埋め込めれば解決するけどその辺りは今後調査。

## Webサービス
「Webサービス」の元の定義は相互運用できるもののはずだったけどもう意味が変わってる。  
https://ja.wikipedia.org/wiki/Web%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9
それでも自分で作ってサービスを名乗るなら外部向けにデータは提供する。  
今回のこれなんかサービス内だけでデータ持ってても意味がない。  
https://artisans.kawax.biz/

API/json/更新時間順とAtom Feed/作成時間順を用意してる。  
取得以外のAPIを作るかは未定。

## 一つだけLaravelネタ
WebComponentsで画像表示して気付いたけどServer PushでLinkヘッダー送ってるのに使われてないので警告が出る。  
実害はないけど気持ち悪いので対応。  
画像関連だけルーティングとミドルウェアを変更。  
ついでにキャッシュ用のヘッダー追加。  
`cache.headers`ミドルウェアとか何気に初めて使った。  
指定できるキーは決まってるのでこういう指定しかできない。  
`'cache.headers:etag;max_age=300;s_maxage=300'`
