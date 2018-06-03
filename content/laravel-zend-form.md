---
title: "Laravel + Zend Form"
date: 2018-06-03T15:26:44+09:00
categories: ["Laravel"]
draft: false
---

Laravel使ってるとフォームのhtml書いてる時が一番面倒。
LaravelCollectiveを分離した辺りでLaravel側としてはここを便利にする気はなくなってるのかな。
https://github.com/LaravelCollective/html
Laravel側はAPIだけでフォームはJSがメインになる想定なのか、結局はhtmlで書くのが一番早いという想定なのか。

どういう想定でも面倒なものは面倒なのでもう少しなんとかできないかとZend Formでフォームのclass化を試してみたらそこそこ上手く行った。
これも複雑なことしようとすると結局手間は変わらないので簡単なフォームが主な対象。

https://docs.zendframework.com/zend-form/

## 使い方

https://github.com/kawax/laravel-zend-form

難しいことはないので軽く。

```
composer require revolution/laravel-zend-form
```

artisanコマンドを用意してるのでmake:formで生成。

```
php artisan make:form SampleForm
```

ファイルの置き場所は`app/Http/Forms/SampleForm.php`

https://github.com/kawax/laravel-zend-form-project/blob/master/app/Http/Forms/SampleForm.php

Form classはZend Formのドキュメントをよく見て中身を組み立てる。
https://docs.zendframework.com/zend-form/quick-start/#factory-backed-form-extension

使う時はコントローラーで

```php
use App\Http\Forms\SampleForm;

    public function __invoke()
    {
        $form = new SampleForm;

        return view('form')->with(compact('form'));
    }
```

DIでもいいけど

```php
use App\Http\Forms\SampleForm;

    public function __invoke(SampleForm $form)
    {
        return view('form')->with(compact('form'));
    }
```

ビューではこれだけで済むのが理想。

```php
{{ $form->render() }}
```

ZendFormの生成するhtmlでいいならこれでいいけど実際はそう簡単じゃない。
一応CustomElement作るとかForm class側でがんばってやれないことはないけどそこまでの手間をかけるかどうか。

表示を細かく制御したいなら各項目ごとに表示。これをやるなら普通に作っても同じ気がする。

```php
@php
    $form->prepare();
@endphp

{!! $form->form()->openTag($form) !!}

{{ csrf_field() }}

<div class="form-group row">
    <label for="name" class="col-sm-3 col-form-label">{!! $form->get('name')->getLabel() !!}</label>
    <div class="col-sm-9">
        {!! $form->formInput($form->get('name')) !!}
    </div>
</div>

<div class="form-group row">
    <label for="email" class="col-sm-3 col-form-label">{!! $form->get('email')->getLabel() !!}</label>
    <div class="col-sm-9">
        {!! $form->formInput($form->get('email')) !!}
    </div>
</div>

<div class="form-group row">
    <div class="col-sm-9 offset-sm-3">
        {!! $form->formSubmit($form->get('send')) !!}
    </div>
</div>

{!! $form->form()->closeTag($form) !!}
```

簡単なフォームにだけ使うとか、formCollectionで一部の項目にだけ使うとか、どう使うかはその場の判断。

## 詳細
パッケージの中身。今回はこれしかないけど。
https://github.com/kawax/laravel-zend-form/blob/master/src/Form.php

Renderer部分が一番分からなかった。LaravelもだけどView関連を他で使おうとすると大変。

```php
    /**
     * @return PhpRenderer
     */
    protected function getRenderer(): PhpRenderer
    {
        if (is_null($this->renderer)) {
            $this->renderer = new PhpRenderer;
            $configProvider = new ConfigProvider;
            $pluginManager = new HelperPluginManager(new ServiceManager, $configProvider()['view_helpers']);
            $this->renderer->setHelperPluginManager($pluginManager);
        }
        return $this->renderer;
    }
```

これを自力で解決するのは時間かかっただろうけどこの記事を見つけたのですぐ終わった。
https://igotaprinter.com/blog/zf3-forms-standalone-plus-mustache-templates.html


`{{ }}`でエスケープされないのは`Illuminate\Support\HtmlString`だから。
有効なのはrender()だけなので他は`{!! !!}`を使う必要がある。

```php
     /**
     * @return HtmlString
     */
    public function render(): HtmlString
    {
        return new HtmlString($this->getRenderer()->form($this));
    }
```

ViewHelperはマジックメソッドで。

```php
public function __call($method, $arguments)
{
    $renderer = $this->getRenderer();
    if (is_callable([$renderer, $method])) {
        return call_user_func_array([$renderer, $method], $arguments);
    }
    throw new \BadMethodCallException(sprintf('Method [%s] does not exist.', $method));
}
```

途中で見つけたこれはHelperを自分で一つ一つ作ってるけどさすがにそんなことはしたくない。
https://github.com/spotonlive/sl-laravel-zf2-form
FacadeでのHelperもやめた。
Formオブジェクトから直接呼び出せば十分。

Zendでのこれを

```php
echo $this->formInput($form->get('name'));
```

こう書けるのでZendの情報を参考にできる。

```php
{!! $form->formInput($form->get('name')) !!}
```

ViewHelperも大分ややこしい。
formから
https://github.com/zendframework/zend-form/blob/master/src/View/Helper/AbstractHelper.php
n18nに行って
https://github.com/zendframework/zend-i18n/blob/master/src/View/Helper/AbstractTranslatorHelper.php
viewまで
https://github.com/zendframework/zend-view/blob/master/src/Helper/AbstractHelper.php

Form使うだけでもあちこちに依存している。
composer.jsonではrequireで指定してないので追加でインストールが必要。

久しぶりにZend見たけど設計の思想からしてよく分からない。
いやまぁLaravelが特殊なだけでZendくらいが普通ではあるけど。

## バリデーション
Laravel側の機能使えばいいのでFormでは完全に無視。