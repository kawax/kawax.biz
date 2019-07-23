---
title: "AWS/EFS リージョン間データ転送"
date: 2019-07-23T23:46:13+09:00
categories: ["AWS"]
draft: false
---

DataSync使えば転送できるけどわざわざ手動で設定する必要はない。  
https://docs.aws.amazon.com/ja_jp/datasync/latest/userguide/what-is-datasync.html

AWSが用意してるツール使ったほうが簡単・確実。実体はCloudFormationで必要なAWSリソースを作成する。  
https://github.com/aws-samples/amazon-efs-tutorial/tree/master/in-cloud-transfer

使い方は簡単なので詳細は省略。

（未確定情報）VPCピアリングが必要かもしれない。VPCのCIDRが同じだとエラーとか色々あるので必要なら新規VPCを作る。

- 転送元バージニアリージョン→転送先東京リージョン
- 転送先に新規EFSを作る。（VPCピアリング使う場合マウントターゲットは新VPCのほうで設定）
- in-cloud-transferは転送元リージョンのDeploy to AWSを選択。
- パラメータを入力して進める。EC2インスタンスタイプはc5など指定のものしか使えないので無駄に起動したままにしないように注意。不要になったら終了していい。
- 転送元。Systems Manager→メンテナンスウィンドウ→アクションからメンテナス時間の編集→CRON/Rate式。ここで設定した時間に転送されるので一度だけ実行されるように設定。タイムゾーンはUTC。
- 転送先。DataSyncのページで転送状態を確認。ファイル数が多いとかなり時間がかかるはず。
- 転送が完了したらマウントターゲットのVPCを変更。
- 全部終わったらCloudFormationのin-cloud-transferを削除してEC2などを削除。DataSync関連は消えてないので手動で削除。

この後マウントして必要ならパーミッションなどの変更。これも時間かかる。  
いきなり数百GBのEFSを転送しようとせず小さいEFSで練習したほうがいい。