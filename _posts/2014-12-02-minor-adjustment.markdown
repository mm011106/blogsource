---
layout: post
title: "Octopress+OCT2のちょっとした修正"
date: 2014-12-02 20:28:23+0900
comments: true
categories: octopress, oct2
---


今まで、ブログのポスト日時表示がうまく行かなかったので色々と調べまわっていました。しかし、web上ではなかなか情報がなく、半分諦めていました。ふと、昔はどうだったかなあ、と気になってoctopressをインストールしたばかりの時に試しに使っていたテーマで生成したHTMLコードを見てみたところ、ブログページに日付が入っていることを発見。なら、この部分だけ持ってきて入れなおしてみるかな、とやってみたところうまく行きました！

<!-- more -->

このページはOCT2というテーマを使っているのですが、そのソース`source/_include/date.html`では日付が代入されている（と思われる）変数が`post.date_formatted`となっているのですが、これではうまく表示できないようです。ポスト日時が表示できていた昔のソースではここが`post.date`になっていました。  
{% codeblock %}
{% raw %}

{% capture date %}{{ page.date }}{{ post.date }}{% endcapture %}
{% capture date_text %}{{ page.date_text }}{{ post.date_text }}{% endcapture %}
{% capture has_date %}{{ date | size }}{% endcapture %}

{% capture updated %}{{ page.updated }}{{ post.updated }}{% endcapture %}
{% capture updated_text %}{{ page.updated_text }}{{ post.updated_text }}{% endcapture %}
{% capture was_updated %}{{ updated | size }}{% endcapture %}

{% if has_date != '0' %}
  {% capture time %}<time datetime="{{ date | datetime | date_to_xmlschema }}" pubdate{% if updated %} data-updated="true"{% endif %}>{{ date_text }}</time>{% endcapture %}
{% endif %}

{% if was_updated != '0' %}
  {% capture updated %}<time datetime="{{ updated | datetime | date_to_xmlschema }}" class="updated">Updated {{ updated_text }}</time>{% endcapture %}
{% else %}{% assign updated = false %}{% endif %}

{% endraw %}
{% endcodeblock %}
中身がわからないまま使っているので、この変数がどこでどのように定義されているのかいまいちわかっていませんが、`post.date_formatted`にはうまく値が入っていないようです。あるいは、`formatted`にするために何か他のモジュールのようなものがいるのかもしれません。  
やったことは、もともとある`octopress/_include/date.html`のファイル名を変更してオリジナルを保存。次に、うまく時間表示ができていた以前のテーマの同じテンプレートをコピー。それだけ。

あまり深く詮索せず、とりえあず表示できたので良しとします。日付のフォーマットも西洋風ですが、まああまり違和感ないのでこのまま。

