---
title: avanpo
home: active
---

{% for post in site.posts limit:1 %}
<p class="date">{{ post.date | date_to_string }}</p>
<h1><a href="{{ post.url }}">{{ post.title }}</a></h1>

{{ post.content }}
{% endfor %}
