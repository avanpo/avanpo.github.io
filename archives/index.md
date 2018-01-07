---
title: "archives :: nullset"
archives: active
---

# Archives

<ul class="archives">
{% for post in site.posts %}
  <li><span class="date">{{ post.date | date_to_string }}</span> &rarr; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
