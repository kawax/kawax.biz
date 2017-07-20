---
title: "マストドン"
date: 2017-07-23T22:31:43+09:00
categories: [""]
draft: false
---


## 構成

- AWS
- EC2 Container Service (Docker)
- RDS (PostgreSQL)
- ElastiCache (Redis)
- S3
- CloudFront
- ALB
- SES


## 運用
GitLabに置いてローカルからgit push→GitLab CI→AWSへデプロイで簡単にアップデート。  
マストドンはアップデートしやすい環境を整えておくのが大事。


## コントリビュート
マストドン本体にもプルリクしたいけどRailsはあまり使わない、Reactはちょっとできる程度なので機能追加は1回だけ。

