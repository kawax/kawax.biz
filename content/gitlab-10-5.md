---
title: "GitLab 10.5"
date: 2018-02-24T11:04:45+09:00
categories: ["GitLab"]
draft: false
---

GitLab 10.5 released with Let's Encrypt integration, Gemnasium dependency checks, and CI/CD external files | GitLab
https://about.gitlab.com/2018/02/22/gitlab-10-5-released/

完全に非公開なのでhttpのままだったけどLet's Encrypt対応で簡単にできるようになったのでhttpsにした。

`/etc/gitlab/gitlab.rb`に追記して`gitlab-ctl reconfigure`
```ruby
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['foo@email.com'] # Optional
```
cronで自動更新
```bash
0 0 * * * /opt/gitlab/bin/gitlab-ctl renew-le-certs > /dev/null
```

GitLabのアップデートは99％は`sudo yum update -y`だけで問題なく終わるけどまれに障害が発生する。
今回はhttpsにしたことでCIが失敗するようになった。
原因はLet's Encryptなのに自己証明書扱いでこのエラー `x509: certificate signed by unknown authority`
検索すると同様の事例は多かったのでよくあるんだろう。

色々試したけど最終的にはそもそも検証を無効化して対応。
```bash
git config --global http.sslverify false
```
GitLab内で使うだけなのでまぁいいかと。

verifyが成功すれば完了。
```bash
sudo gitlab-runner verify
```
