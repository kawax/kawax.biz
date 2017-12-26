---
title: "Laravel Macroable"
date: 2017-12-25T11:58:41+09:00
categories: ["Laravel"]
draft: false
---

`Manager` の説明は難しいけど `Macroable` は簡単なのでまとめておく。  
中身は100行くらいしかなくて見れば分かるくらい簡単。  
`macro()` で登録したメソッドをマジックメソッドで呼び出す。  
https://github.com/laravel/framework/blob/8d0508989deb01e18ffa9d216674117416daee2f/src/Illuminate/Support/Traits/Macroable.php

Manager はどうせなら他の人が拡張できるようなパッケージを作りたいけどいいアイデアが思い付かない。  
公開してないアプリ内ではよく使ってるけど汎用性ないので。

## Laravel内
使ってる例としては Collection。  
https://readouble.com/laravel/5.5/ja/collections.html#extending-collections

全部は把握してないけど他にも色々な所で有効なはず。

## 自作クラス内で使う

これだけ。

```php
namespace App;

use Illuminate\Support\Traits\Macroable;

class Foo
{
   use Macroable;
}
```

ServiceProvider で登録。

```php
Foo::macro('my', function() {
    return 'my';
});
```

どこかで使用。

```php
echo Foo::my();
echo (new Foo)->my();
```

自作クラス内でもマジックメソッド使ってる場合は `Macroable` のと合わせて上手く処理する。

```php
    public function __call($method, $parameters)
    {
        //自作クラス用の__call
        
        
        //Macroable
        if (static::hasMacro($method)) {
            if (static::$macros[$method] instanceof \Closure) {
                return call_user_func_array(static::$macros[$method]->bindTo($this, static::class), $parameters);
            }
        }

        //どっちにもなかった場合
        throw new \BadMethodCallException(sprintf('Method [%s] does not exist.', $method));
    }
```

もしくはこの方法  
https://github.com/laravel/framework/blob/a55c4bf892d4cdcd0213218fe77076c4a1b6969c/src/Illuminate/Support/Optional.php

```php
    use Macroable {
        __call as macroCall;
    }
    
    public function __call($method, $parameters)
    {
        if (static::hasMacro($method)) {
            return $this->macroCall($method, $parameters);
        }
        
        //
    }
```

自分のアプリ内だけで使うならこんなことする必要はなくて、これが便利なのは他の人も使う場合。

## 自作パッケージで使う

composer.json に
```json
"illuminate/support": "5.*",
```
と指定しておけば `Illuminate\Support\Traits\Macroable` が使える。

使う側が好きなように拡張できる。

Macroable に Laravel 依存な部分はないので Laravel 以外でも使える。
