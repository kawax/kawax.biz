---
title: "Mastodon"
date: 2017-07-23T22:31:43+09:00
categories: ["Mastodon"]
draft: false
---

2017年4月頃シュッと立てた。

### 構成

- AWS
- EC2 Container Service (Docker)
- RDS (PostgreSQL)
- S3
- CloudFront
- ALB
- SES

### 運用
GitLabに置いてローカルからgit push→GitLab CI→AWSへデプロイで簡単にアップデート。マストドンはアップデートしやすい環境を整えておくのが大事。

マストドン運営できるのは最低限のスキル持ってる証拠として分かりやすい。

### コントリビュート
マストドン本体にもプルリクしたいけどRailsはあまり使わない、Reactはちょっとできる程度なので機能追加は1回だけ。

### Laravel用ライブラリ
自分で使うためMastodon APIライブラリを作った。

- https://packagist.org/packages/revolution/laravel-mastodon-api
- https://packagist.org/packages/revolution/socialite-mastodon

### tootlog
マストドンはどんどんサーバー消える想定なのでログを残すためのサービスを作った。  
https://tootlog.com/

- Laravel
- Vue.js
- ElasticBeanstalk
