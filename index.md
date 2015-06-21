---
title: avanpo
home: active
---

## Blog Posts

{% for post in site.posts %}
* <span class="date">{{ post.date | date_to_string }}</span> &rarr; [{{ post.title }}]({{ post.url }})
{% endfor %}

## Misc

Projects, etc
