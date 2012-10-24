---
layout: page
title: Yi Wang's Tech Notes
tagline: Supporting tagline
---
{% include JB/setup %}

I had a [tech blog](http://cxwangyi.wordpress.com/) with the same name
but hosted on wordpress.com.  However Chinese readers cannot access it
since wordpress.com was blocked in China.  Fortunately, Github has not
been blocked yet.  So I set up this blog using Jekyll and Github.

I have a plan to port my posts on the wordpress.com blog to here.

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
