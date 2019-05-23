---
title: "AWS CodeStarで後からgitリポジトリを変更する"
date: 2019-05-23T21:31:01+09:00
categories: ["AWS"]
draft: false
---

DevOps環境を一発で用意してくれるCodeStar。詳細はここでは省略。
https://aws.amazon.com/jp/codestar/

GitHubかCodeCommitにリポジトリも作成するけどこの時既存リポジトリは選択できない。
必ず新規に作成するしかない。
選択できても色々と設定ファイルがないとデプロイできないので仕方ない仕様。
先に設定ファイルを準備すれば変更はできる。

## 既存リポジトリ側の準備
新規に作成したリポジトリから設定ファイルを持ってくる。
例えばLaravelのプロジェクトテンプレートで作るとこれが作成される。Laravel5.3なので古いけど普段見ない設定ファイルがあるのが分かる。
https://github.com/kawax/codestar-php-laravel

- appspec.yml : CodeDeploy用のファイル。
- scripts/ : appspec.ymlから実行してるスクリプト。
- buildspec.yml : CodeBuild用のファイル。template-configuration.jsonはここで必要。
- template.yml : CloudFormation用のファイル。

必要ならそれぞれ変更。

## CodeStarの変更
実際にはCodePipelineの設定。

なお、設定変更するにはIAMの権限が必要。管理者以外でエラーが出た場合は権限の設定してもらうしかない。
CodeStar使おうとするとこれ以外にも色々権限が必要になる。
https://dev.classmethod.jp/cloud/aws/iam-pass-role/

CodeStarダッシュボードからスタート
↓
「継続的デプロイメント」タイル
↓
「パイプラインの編集」
↓
CodePipeline。一番上「編集する：Source」部分の「ステージを編集する」
↓
えんぴつマーク？っぽい編集ボタン
↓
「GitHubに接続する」から既存リポジトリとブランチを選択。
↓
保存していって最終的に「変更をリリースする」。

設定ファイルを間違えてなければデプロイされるはず。

## 終わり
CodeStarは今調査してるので色々あるけど今回はこれだけ。

自分で見るならCodeStar使う必要はないけど
他社のAWSで自分がずっと見るわけでもなく詳しい人がいない状態の場合、CodeStarから始めるのがたぶん一番簡単。
ブラックボックスな独自AMIでPHPのバージョンアップもできないような状態ではどうにもならない。

CodeStarから始めるとDevOps環境一式を全部用意してくれる。
AMIは素のAmazon Linux。PHPのインストールなどはスクリプトでやってるのでバージョンアップでもなんでもできる。
AWSリソースもCloudFormationで設定。
ほとんどのことをリポジトリに含める設定ファイルで管理できる。

他社のAWSを一から調査するとどこで何を使ってるかが分かりにくいのでこれだけ見ればいいと言えると後で楽。