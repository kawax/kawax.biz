---
title: "AWSサービスのみでマストドンのデプロイ"
date: 2018-01-09T00:02:55+09:00
categories: ["Mastodon", "AWS"]
draft: true
---

GitLab→ecs-cliでECSへという状態からAWSのサービスのみを使ったデプロイへ。

## 構成
CodeCommit→CodeBuild→CodePipeline→ECS

ECS以外初めて使う所からの設定。今は全部東京リージョンで使える。

## CodeCommit
gitリポジトリサービス。

- 色々セットアップ https://docs.aws.amazon.com/ja_jp/codecommit/latest/userguide/setting-up-ssh-unixes.html
- IAMユーザーに SSH Key のアップロード。

SSH接続の場合、最終的にはこれが表示できれば成功。HTTPS接続はドキュメントの他のページ参照。

```
ssh git-codecommit.ap-northeast-1.amazonaws.com
You have successfully authenticated over SSH. You can use Git to interact with AWS CodeCommit. Interactive shells are not supported.Connection to git-codecommit.ap-northeast-1.amazonaws.com closed by remote host.
Connection to git-codecommit.ap-northeast-1.amazonaws.com closed.
```

git cloneもpushもできるようになる。

```
git clone https://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/...
```

マストドンのコードをpushして表示される所まで確認。

## CodeBuild
- ここはこの記事で十分だった。 https://dev.classmethod.jp/cloud/aws/codepipeline-support-ecs-deploy/
- CodeBuild が管理してる Docker 環境なのでサーバーのことは気にしなくていい。

`buildspec.yml` は GitLab CI でやってたことと大体同じ。 
 
- `docker-compose` はインストール済なのでそのまま使える。
- `aws ecr get-login` には少し前に `--no-include-email` が必要になった。
- `$CODEBUILD_RESOLVED_SOURCE_VERSION` は手動でビルドテスト時には空なので `$CODEBUILD_BUILD_ID` に一旦変更。後で元に戻す。

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws --version
      - docker-compose build
      - docker-compose run --rm web rails assets:precompile
      - docker-compose run --rm web rails db:migrate
      - $(aws ecr get-login --no-include-email --region ${AWS_DEFAULT_REGION})
      - REPOSITORY_URI=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_NAME}
      - IMAGE_TAG=$(echo $CODEBUILD_BUILD_ID | cut -c 1-7)
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - echo "[{\"name\":\"${IMAGE_NAME}\",\"imageUri\":\"${REPOSITORY_URI}:${IMAGE_TAG}\"}]" > imagedefinitions.json
artifacts:
    files: imagedefinitions.json

```

ビルドプロジェクトの環境変数で `AWS_ACCOUNT_ID` `IMAGE_NAME` `AWS_DEFAULT_REGION` を設定。
クラスメソッドの記事はたぶん `AWS_DEFAULT_REGION` のことを書き忘れてる。
`IMAGE_NAME` は ECR のリポジトリ名。

docker-compose でビルドして assets:precompile してそこからさらに docker でビルドしてるので GitLab でやると結構重い。CodeBuild での処理に分離できるのが最大のメリット。

CodeBuild は Docker image のビルドとECRへのアップロードまで。
手動でビルドして成功したら次へ。

## CodePipeline
CodeCommit と CodeBuild を合わせてデプロイしてくれるのが CodePipeline。

- Source: CodeCommit
- Build: CodeBuild
- Deploy: ECS

今回はAWSのみにするためにこうしてるけど組み合わせは色々変更できる。

順番に設定するだけなので難しい所はない。

## マストドン用に調整
サンプルレベルのアプリならドキュメント通りでも問題なく動くけどマストドンの規模だと無理だった。
`buildspec.yml` の `imagedefinitions.json` がタスク定義なのでこれをもっと詳細に書く必要がありそう。
ecs-cli なら `docker-compose.yml` に書いてる内容をJSONで作り直し。
それだけならいいけど環境変数の扱いで面倒そうなので今回は CodePipeline から ECS へのデプロイを諦める。
CodeBuild の段階で ecs-cli を使ってデプロイ。

最終的な `buildspec.yml`。これだけ見ても参考にならないけど。

```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws --version
      - aws s3 cp $S3_ENV .env.production
      - curl https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-linux-amd64-latest -o ecs-cli
      - chmod +x ./ecs-cli
      - ./ecs-cli -v
      - docker-compose build
      - docker-compose run --rm web rails assets:precompile
      - $(aws ecr get-login --no-include-email --region ${AWS_DEFAULT_REGION})
      - REPOSITORY_URI=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_NAME}
      - IMAGE_TAG=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - echo "[{\"name\":\"${IMAGE_NAME}\",\"imageUri\":\"${REPOSITORY_URI}:${IMAGE_TAG}\"}]" > imagedefinitions.json
      - ./ecs-cli compose -c $ECS_CLUSTER -f aws-migrate.yml --project-name ecscompose-service-mastodon-migrate up
      - ./ecs-cli compose -c $ECS_CLUSTER -f aws-web.yml --project-name ecscompose-service-mastodon-web service up --timeout 10
      - ./ecs-cli compose -c $ECS_CLUSTER -f aws-api.yml --project-name ecscompose-service-mastodon-api service up --timeout 10
artifacts:
    files: imagedefinitions.json

```
### CodePipeline を使う場合のメモ
マストドンだとwebとapiの2つをECSサービスで動かしてるので最後の Deploy 部分で並列に実行する。パイプライン作成時にはできないので一度作ってから変更。

GitLabでは `db:migrate` はECSの一度のみのタスクとして実行していたけど CodePipeline では無理そうなので `buildspec.yml` に書いて CodeBuild で実行。


## ひとまず終わり
結局はGitLabの内GitリポジトリとCI機能をAWSのサービス使う形になっただけ。
CodePipeline の ECS 対応を活かせてないので Fargate が東京リージョンが来たらまた試す。
