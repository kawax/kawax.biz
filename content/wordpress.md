---
title: "[WIP]WordPress"
date: 2017-07-25T22:27:40+09:00
categories: ["AWS", "WordPress"]
draft: false
---

## AWSで大量のWordPressを動かす

`EC2 - ELB - CloudFront - Route53`

1台のEC2で100以上のWordPress動かしてる…。

### セキュリティ
- Wordfence
- Crazy Bone
- WordPress本体もプラグインもテーマも常時自動アップデート
- ログインURL変更
- CloudFront + WAF

実際の所WordPress本体やプラグインが原因で不正アクセス成功されたことはない。ほとんどは簡単なパスワード使ってる管理者が原因。99%は人災。

### SSL
AWS Certificate Managerで対応可。

### ログ
不正ログイン試行が大量に来るのでアクセスログはCloudWatch Logsで記録。

### サーバー監視
CloudWatchやMackerel

### 通知
色々な通知先はChatWork
