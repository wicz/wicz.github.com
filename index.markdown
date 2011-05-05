---
layout: default
top: Vinicius Horewicz
---

Blog Posts ![Feed icon](/images/feed-icon-14x14.png)
==========

{% for post in site.posts %}
<div class="section list">
  <div class="date">{{ post.date | date_to_string }}</div>
  <p class="line">
    <h2><a class="title" href="{{ post.url }}">{{ post.title }}</a></h2>
    <a class="comments" href="{{ post.url }}#disqus_thread">View Comments</a>
  </p>
  <p class="excerpt">{{ post.excerpt }}</p>
</div>
{% endfor %}

<script type="text/javascript">
//<![CDATA[
(function() {
		var links = document.getElementsByTagName('a');
		var query = '?';
		for(var i = 0; i < links.length; i++) {
			if(links[i].href.indexOf('#disqus_thread') >= 0) {
				query += 'url' + i + '=' + encodeURIComponent(links[i].href) + '&';
			}
		}
		document.write('<script type="text/javascript" src="http://disqus.com/forums/wicz/get_num_replies.js' + query + '"></' + 'script>');
	})();
//]]>
</script>
