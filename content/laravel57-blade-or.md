---
title: "Laravel5.7 で Blade の or が削除される"
date: 2018-03-28T11:49:00+09:00
categories: ["Laravel"]
draft: false
---

Update Your Blade Templates to Use the Null Coalesce Operator - Laravel News
https://laravel-news.com/blade-templates-null-coalesce-operator

Laravelはばっさり行くよな…。PHP7なら不要とはいえこんなの互換のために残せばいいのに。

```php
{{ $name or 'Guest' }}
```

こう変更。
```php
{{ $name ?? 'Guest' }}
```

非推奨にして移行期間設けるわけでもなく半年後の次期バージョンで即削除。
仕方ないので今の内から修正作業していくしかない。


- 急に大きな変更されたら困る→LTS版使え。
- 0.0.1のマイナーバージョンではBREAKING CHANGEは絶対に含めない。
- その代わり0.1のメジャーバージョンでは遠慮なく破壊する。

最近はこんな方針。
