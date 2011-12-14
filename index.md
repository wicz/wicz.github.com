---
layout: default
top: Vinicius Horewicz
---

<div id="blog-archives" class="category">
  {% for post in site.posts %}
    <article>
      <h2><a href="{{ root_url }}{{ post.url }}">{{post.title}}</a></h2>
      <time datetime="{{ post.date | datetime | date_to_xmlschema }}" pubdate>{{ post.date | date_to_string }}</time>
    </article>
  {% endfor %}
</div>