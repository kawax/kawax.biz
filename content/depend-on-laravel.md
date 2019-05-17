---
title: "どこまでLaravelに依存するか"
date: 2019-05-17T22:26:03+09:00
categories: ["Laravel"]
draft: false
---

よく迷うのはコントローラーから別のクラスに処理をぶん投げる時に`$request`を渡すかどうか。

```php
    public function __invoke(Request $request, PostService $service)
    {
        $post = $service->store($request);

        return $post;
    }
```

渡した先では$request使って色々。

```php
$post = $request->user()
                ->posts()
                ->create($request->only(['title', 'message']));
```

これはLaravelのRequestにおもいっきり依存している。

依存しないように作ると`$request->user()`でもUserモデルなので`$request->user()->id`でidだけ渡す。
渡した先でEloquent使ったらまたLaravelに依存なのでUserRepository使う形にして…とか考えてると何の意味があるんだとなる。

これこそ設計の話で世間では色々出てるけどLaravelに限ると納得できる話は少ない。
テストを基準にするとインメモリSQLiteでDB使ったテストすればいい派なので依存しててもテストはできる。
Laravelの便利な機能はすべてテストのためと言えるくらいLaravelは元からテストしやすい。

Laravel以外の常識に囚われて過剰な設計してる例が多い。

結局はプロジェクト次第という結論しかないのでもう終わり。