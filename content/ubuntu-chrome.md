---
title: "Laravel Dusk用 Ubuntu18.04にChromeをインストール"
date: 2018-12-22T11:05:35+09:00
categories: ["Laravel"]
draft: false
---

```
curl https://dl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | sudo tee /etc/apt/sources.list.d/google-chrome.list
sudo apt update
sudo apt install google-chrome-stable
```

日本語フォント
```
sudo apt install fonts-ipafont-gothic
```

ChromeDriverはDuskに含まれてるのでLaravel Console Duskと合わせると本番環境でDuckが使える。
https://github.com/nunomaduro/laravel-console-dusk
