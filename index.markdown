---
layout: default
top: Vinicius Horewicz
---

{% for post in site.posts %}
<div class="section list">
  <div class="date">{{ post.date | date_to_string }}</div>
  <p class="line">
    <h2><a class="title" href="{{ post.url }}">{{ post.title }}</a></h2>
  </p>
  <p class="excerpt">{{ post.excerpt }}</p>
</div>
{% endfor %}
