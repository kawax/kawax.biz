---
title: "Laravel8でapp/Modelsディレクトリが復活"
date: 2020-08-22T08:08:59+09:00
categories: ["Laravel"]
draft: false
---

https://github.com/laravel/laravel/commit/710d472d764983a0f8074b6b1a3c5cf57adce1e4

Laravel4では`app/models`が存在したけど5.0で消えた。
「モデルはapp直下に置くのがLaravelのルール」なんて嘘書いてる人は多かったけど違う。
「モデルの置き場所は自分で決めろ」って方針。 https://readouble.com/laravel/7.x/ja/structure.html
app直下にUser.phpとかPost.phpとかどんどん増えていったらおかしいと気付くだろう。

Laravel8で復活したのはTwitterの投票で`app`派と`app/Models`派どっちが多いか聞いた結果らしい。

自分は`app/Model`を作っていた。Laravelデフォルトのディレクトリは複数形なので自分で作ったディレクトリは単数形にして区別するため。`Models`に変えよう。