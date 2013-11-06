---
layout: index
top: Vinicius Horewicz
---

<header>
  <h1>Vinicius Horewicz</h1>
  {% include links.html %}
</header>

<section class="blog-posts">
  {% for post in site.posts %}
    <article>
      <time datetime="{{ post.date | datetime | date_to_xmlschema }}" pubdate>{{ post.date | date_to_string }}</time>
      <h3><a href="{{ root_url }}{{ post.url }}">{{post.title}}</a></h3>
    </article>
  {% endfor %}
</section>

