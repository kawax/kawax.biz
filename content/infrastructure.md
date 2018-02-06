---
title: "AWS 最近のインフラ構成"
date: 2018-02-06T13:42:36+09:00
categories: ["AWS"]
draft: false
---


## 小規模版
AWSと言ってもなんでも大規模な運用するわけではないので1サーバーに複数サイト入れてる程度のコスト優先な場合。


まずEC2+RDSが基本。本当に個人レベルならRDSは使いたくないけどそれ以外はDBだけは必ず分離したい。もちろんRDSは複数サイトで使い回す。コスト的にはRDSが一番高い。
DB以外にファイルも扱うならS3。
AWSの基本形。

EC2のサーバーセットアップにはLaravel Forge。LaravelだけでなくWordPress動かしても高速でAMIMOTOとかKUSANAGIとかいらなかったんだ…となってる。今更手作業でセットアップなんてしないのでこの規模ならForgeで省略。

肝心のアプリのデプロイもForge。GitHub/GitLab.com/Bitbucket/Self hosted GitLabなどからPush to deploy。今はこれができないとやってられない。

後はELB下にEC2配置してRoute53の設定すればウェブサイトとしては公開できる。サイト数が少なければELBなしでもいいけど多くなると管理の手間が色々増えるので使う。
ちなみにRoute53の管理にはroadworker。とはいえ使える人ばかりではないのでバックアップとしてしか使えてない。たまに大規模に書き換える時はroadworkerでまとめて作業。

SSL対応はACM。ELB使わないならForgeでLet's Encrypt。
ACMをELBかCloudFrontに付けるだけ。

前はACMとWAFのためにCloudFront使ってたけどELB(ALB)ならどっちも対応してるのでCloudFront使うかは必要に応じて。

サーバー増やして分散したい場合はForgeでセットアップしてELBに配置すればいい。
手作業になるけど2,3サーバー程度なら問題ない。それ以上になったら小規模の範囲じゃない。

日々のメンテ。Ubuntuなのでセキュリティアップデートは自動。必要な作業があればForgeのRecipeで。

コスト面はリザーブドインスタンスで安く。

小規模だとこのくらいしか書くことない。
いかに自動化して効率よくするかしか考えてない。けどCloudFormation使うほどではないという規模感。
ACMなどaws-cliツールで自動化できる所はしている。


## 少し規模が大きい版
Docker使う場合。と言っても普通にECS(Elastic Container Service)使ってるだけなので特に書くことはなかったり。
マストドンがちょうどいい練習になる。

ECSだとスポットインスタンス使えるので必要なEC2数は増えるけど意外と安い。
