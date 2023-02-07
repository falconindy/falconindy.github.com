---
title: Articles
layout: page
---
<ul class="posts">{% for post in site.categories.articles %}
  <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}</ul>

