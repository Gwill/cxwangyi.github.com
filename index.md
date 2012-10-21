---
layout: page
title: 王益的技术笔记
tagline: Supporting tagline
---
{% include JB/setup %}

我有一个wordpress.com上的[技术博客](http://cxwangyi.wordpress.com/)，可
是wordpress不幸上了黑名单，以至于想看看自己的博客还得借助高科技，比如
Google Reader。在同事彪哥的建议下，用Jekyll和Github搭建了这个博客。

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
