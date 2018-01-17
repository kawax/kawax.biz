---
title: "Lambda+Golang版 CloudWatchアラームをChatWorkに通知"
date: 2018-01-17T09:24:33+09:00
categories: ["golang"]
draft: false
---

たぶんみんな作るので自分で作るには初日にやるしかない。
SNSイベントを受け取って ChatWork に投稿するだけなので簡単。
実際 Golang での開発部分よりその後のデプロイ部分で試行錯誤した時間のほうが長いかも。
この前使った CodeBuild と CodePipeline の経験が早速役に立った。

https://github.com/kawax/lambda-alarm-chatwork

## 注意点
公式のドキュメントが間違ってる。  
https://github.com/aws/aws-lambda-go/blob/master/events/README_SNS.md

```go
import (
    "strings"
    "github.com/aws/aws-lambda-go/events")

func handler(ctx context.Context, events.SnsEvent snsEvent) {
    for _, record := range snsEvent.Records {
        snsRecord := record.Sns

        fmt.Printf("[%s %s] Message = %s \n", record.EventSource, snsRecord.Timestamp, snsRecord.Message) 
    }
}
```

引数部分が逆。
`snsEvent events.SnsEvent`

大文字。
`record.SNS`

戻り値返さないとLambdaでは失敗扱いになるかも。  
https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/go-programming-model-handler-types.html
