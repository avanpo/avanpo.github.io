---
title: avanpo
home: active
---

{% for post in site.posts limit:1 %}
<h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
<p class="date">{{ post.date | date_to_string }}</p>

{{ post.content }}
{% endfor %}
