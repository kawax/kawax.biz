---
title: "Mastodon"
date: 2017-07-23T22:31:43+09:00
categories: ["Mastodon"]
draft: false
---

### 構成

- AWS
- EC2 Container Service
- RDS
- S3
- CloudFront
- ALB
- SES

### 運用
GitLabに置いてローカルからgit push→GitLab CI→AWSへデプロイで簡単にアップデート。  
マストドンはアップデートしやすい環境を整えておくのが大事。

### Laravel用ライブラリ
自分で使うためMastodon API使うためのライブラリを作った。

- https://packagist.org/packages/revolution/laravel-mastodon-api
- https://packagist.org/packages/revolution/socialite-mastodon

### コントリビュート
マストドン本体にもプルリクしたいけどRailsはあまり使わない、Reactはちょっとできる程度なので機能追加は1回だけ。
