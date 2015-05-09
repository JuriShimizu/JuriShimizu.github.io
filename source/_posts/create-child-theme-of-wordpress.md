title: WordPressの子テーマを作成する
date: 2015-05-10 07:58:28
tags: wordpress
---

わけあって、WordPressを勉強中です。

デザインをカスタマイズするために、子テーマを作ってみます

<!-- more -->

## 子テーマとは？

子テーマを利用すると、親テーマを書き換えずにデザインのカスタマイズができます。

子テーマは独立しているテーマではなく、親テーマに依存していて、

テーマのカスタマイズの管理画面をいじりたい〜なんていうのは、子テーマではできません。

## 子テーマを作る

WordPressの`wp-content／themes`フォルダに新しいフォルダを作ります。

必須のファイルは`style.css`です。

親テーマの`style.css`をコピーして、こんな感じで１番上のコメントを更新します。

```css
/*
Theme Name: name
Author: XXXX
Description: XXXX
Version: 1.0.0
Template: parent theme
*/
```

必須なのは`Theme Name`と`Template`です。

`Template`には親テーマのディレクトリ名を書きます。

コメントなのに、動作に影響するなんて変な感じですね。

コレで、WordPressの管理画面で自分で作った子テーマを選べるようになります。

## 子テーマをいろいろカスタマイズ

#### デザインを変えたい

文字の色などのデザインを変えたい場合は、`style.css`を更新すればOKです

#### テンプレートを変えたい

デザインではなく、表示する項目とかを変えたいときは、

各テンプレートファイル(header.php、footer.php、single.php などなど)を

親テーマのディレクトリから子テーマのディレクトリにコピーしてきて、カスタマイズします。

#### 関数を追加したり、新しいcssを読み込んだりしたい

関数を追加したり、新しいcssを読み込んだりしたい場合は、`functions.php`をカスタマイズします。

このファイルは親テーマから持ってきてはいけません。

親テーマの`functions.php`と子テーマの`functions.php`に同じ関数があると、
こんな感じのエラーになってしまうのです。

```
Fatal error: Cannot redeclare edin_widgets_init() (previously declared in <子テーマのfunctions.php> in <親テーマのfunctions.php>
```

なので、`functions.php`はこんな感じで、追加したいことだけ書きます。

```php
<?php
/**
 * ここに追加したいことを書く
 */
?>
```









