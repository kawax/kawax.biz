---
title: "Fargate"
date: 2018-07-12T10:44:35+09:00
categories: ["AWS"]
draft: false
---

ECSをFargateに変えてみた。元々ecs-cli使っていれば移行は簡単。  
https://kawax.biz/mastodon-aws/

## buildspec.yml
ecs-cliのオプションに`--launch-type FARGATE`追加。ecs-param.ymlも使用。

```yaml
  - ./ecs-cli compose -c $ECS_CLUSTER -f aws-migrate.yml --project-name mastodon-migrate --ecs-params ecs-param-web.yml up --launch-type FARGATE
  - ./ecs-cli compose -c $ECS_CLUSTER -f aws-web.yml --project-name mastodon-web --ecs-params ecs-param-api.yml service up --timeout 10 --target-group-arn $ALB_WEB_ARN --container-name $ALB_WEB_NAME --container-port 3000 --launch-type FARGATE
  - ./ecs-cli compose -c $ECS_CLUSTER -f aws-api.yml --project-name mastodon-api --ecs-params ecs-param-web.yml service up --timeout 10 --target-group-arn $ALB_API_ARN --container-name $ALB_API_NAME --container-port 4000 --launch-type FARGATE

```

## ecs-param.yml
web用の例。FargateはCPUとメモリの組み合わせが決まってるので違う値を設定するとエラー。  
https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/task-cpu-memory-error.html

api用はメモリ増やさないと落ちる。

```yaml
version: 1
task_definition:
  ecs_network_mode: awsvpc
  task_execution_role: ecsTaskExecutionRole
  task_size:
    cpu_limit: 256
    mem_limit: 512
  services:
    web:
      essential: true
run_params:
  network_configuration:
    awsvpc_configuration:
      subnets:
        - subnet-1
        - subnet-2
      security_groups:
        - sg
      assign_public_ip: ENABLED

```

## docker-compose.yml
portsを`3000`での指定から`3000:3000`に変更。`awslogs-stream-prefix`が必要。

```yaml
version: '2'
services:
  web:
    restart: always
    image: ...
    env_file: .env.production
    command: bash -c "rm -f /mastodon/tmp/pids/server.pid; bundle exec rails s -p 3000 -b '0.0.0.0'"
    ports:
      - "3000:3000"
    logging:
      driver: awslogs
      options:
        awslogs-group: "mastodon/web"
        awslogs-region: "ap-northeast-1"
        awslogs-stream-prefix: "web"

```

## ALB
`ターゲットの種類`をinstanceではなくipにする必要があるのでターゲットグループを2つ作り直し。ロードバランサーのリスナーも変更。

## 不要なサービスやEC2を終了
スポットインスタンスとFargateのどっちが安いかは不明。

## 終わり
Fargateでの起動が成功してからまとめれば簡単。実際は何度も失敗してる。  
毎回CodeBuildでデプロイしてたら時間かかりすぎるので手元でecs-cli実行してテストしたほうがいい。
