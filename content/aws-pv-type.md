---
title: "AWS/Lightsail 月間PVとインスタンスタイプの目安"
date: 2018-07-01T14:50:55+09:00
categories: ["AWS"]
draft: false
---

色々調べてみた結果。

| EC2       | Lightsail | PV    | 同時アクセス |
|-----------|-----------|-------|--------------|
| t2.nano   | $3.5      | 5万   | 5            |
| t2.micro  | $5        | 10万  | 10           |
| t2.small  | $10       | 30万  | 20           |
| t2.medium | $20       | 50万  | 40           |
| t2.large  | $40       | 100万 | 80           |

あくまでサーバー1台の場合の目安。  
実際は途中で複数台構成にすると思う。  
nanoやmicroを複数台にするよりは1台をsmallにしたほうがいいはず。  
これ以上の規模なら1~数台でどうこうという話ではなくなるので全く別の話。  
1000万PVくらいまでなら1台でも可能ではあるけど。c4やc5のxlarge辺りを使えば。月の費用は10万円くらい。

RDSのスペックは同じか一つ上辺り。EC2よりRDSのほうでボトルネックになってることも多い。  
EC2はメモリ=同時アクセス=PVで大体決まって分かりやすいけどRDSはそう単純でもない。  
RDSにもEC2のt2と似たクレジットという概念がある。  
重いバッチ処理を毎日してるような場合アクセス以上にクレジットを消費して遅くなる。  
バッチの頻度を下げるなどクレジットが貯まるような調整が必要だけどどうするかは各プロジェクト次第な話。

（追記）Lightsail半額化の反映。
