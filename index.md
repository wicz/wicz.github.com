---
layout: default
top: Vinicius Horewicz
---

{% for post in site.posts %}
<div class="section list">
  <div class="date">{{ post.date | date_to_string }}</div>
  <h2><a class="title" href="{{ post.url }}">{{ post.title }}</a></h2>
</div>
{% endfor %}
