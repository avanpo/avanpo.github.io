---
title: nullset
home: active
---

{% for post in site.posts limit:1 %}
<p class="date">{{ post.date | date_to_string }}</p>
<h1><a href="{{ post.url }}">{{ post.title }}</a></h1>

{% if post.post_author %}
<p class="author">Written by <a href="{{ post.post_author.link }}">{{ post.post_author.name }}</a></p>
{% endif %}

{{ post.content }}
{% endfor %}
