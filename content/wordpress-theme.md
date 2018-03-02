---
title: "古いWordPressテーマの修正作業は大変"
date: 2018-03-02T11:35:06+09:00
categories: ["WordPress"]
draft: false
---

https://wp-update.info/ とは別件で手伝ってる会社のWordPressを新サーバーに移行中。
前に書いたgit+composerを使う形。
まず自分が数個移行して手順をドキュメントにまとめる。
それを見ながら残りは他の人にやってもらってるけどローカルで動作テストしてからと書いてるのに全くやってないので
今はgit cloneしてテーマを修正している段階。
GitLab+ForgeでPush to Deployの環境は完備してるので修正していくだけ。


例えばfunctions.phpにこういうコード。

```php
class TextArea2 extends WP_Widget {
  function TextArea2() { parent::WP_Widget(false, $name = 'テキストR'); }
    function widget($args, $instance) { extract( $args );
    $title = apply_filters( 'widget_example', $instance['title'] );
    $text  = apply_filters( 'widget_example', $instance['text'] );
    $html .= $text;
    echo do_shortcode($html);
 }
    function update($new_instance, $old_instance) { $instance = $old_instance;
    $instance['title']  = trim($new_instance['title']);
    $instance['text']   = trim($new_instance['text']);
    return $instance;
  }
    function form($instance) {
    $title  = esc_attr($instance['title']);
    $text   = esc_attr($instance['text']);
    $html = "<p>\n";
    $html .= "<label for='{$this->get_field_id('title')}'>タイトル:</label>\n";
    $html .= "<input class='widefat' id='{$this->get_field_id('title')}' name='{$this->get_field_name('title')}' type='text' value='{$title}'>\n";
    $html .= "</p>\n<p>\n";
    $html .= "<label for='{$this->get_field_id('text')}'>テキスト:</label>\n";
    $html .= "<textarea rows='3' colls='20' class='widefat' id='{$this->get_field_id('text')}' name='{$this->get_field_name('text')}' type='text' value='{$text}'>{$text}</textarea>\n";
    $html .= "</p>\n";
    echo $html;
    }
}
add_action('widgets_init', create_function('', 'return register_widget("TextArea2");'));
```

- PHP4時代のclassコンストラクタの書き方。
- `{}`を1行で書きたがる変な癖。CSSでもこんなだった。
- extract。逆のcompactはよく使うけどextractは危険…。
- create_function
- そもそもこれ何のためのウィジェットなんだ…。テキストウィジェットでショートコード使うためにわざわざこんな無駄なことしてる…？`add_filter('widget_text', 'do_shortcode');`の1行で済むんだけど…。

class使ってるのはここだけなのでどこかからコピペしてるだけだろう…。
コンストラクタはPHP7.0で非推奨になってPHP8でもまだエラーになるだけらしい。削除でいいのでは…。
ローカルで動かすだけでエラーや警告が出るからすぐ分かるはずなのに今まで直してないのは本当に意味が分からない。

とりあえず自動整形してエラーが出る箇所のみ直す。

```php
    public function __construct()
    {
        parent::__construct(false, $name = 'テキストR');
    }
```

```php
add_action('widgets_init', function () {
    register_widget("TextArea2");
});
```

テーマ全体がこんな調子なので全ファイルに修正が入る。
1回直すだけならいいんだけど
問題はこれを他のテーマにもコピペしてるのでこんなのが大量にある…。
100か200か。
さすがにそんな量を自分がやるのは無理なのでテーマ作った人に責任持ってやってもらう。

さらに他社でもあまり変わらない。
表向きWordPressが得意とか言ってる会社でも実態はこんな初心者レベルだからマジで…。
