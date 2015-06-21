---
title: "archives :: avanpo"
archives: active
---

# Archives

{% for post in site.posts %}
* <span class="date">{{ post.date | date_to_string }}</span> &rarr; [{{ post.title }}]({{ post.url }})
{% endfor %}
