---
layout: page
tagline: Supporting tagline
---

{% include JB/setup %}

{% for post in site.posts limit: 2 %}
<article class="hoverable post">
	<div class="post-header">
	  <h1>{{ post.title }}</h1>
	  <span class="bold">{{ post.date | date: "%B %e, %Y" }}</span> 
	  {% if post.tags.size > 0 %}
	  	|
		  {% for tag in post.tags: %}
		  	<a href="{{ HOME_PATH }}tags.html#{{ tag }}-ref"><span class="tag">#{{ tag }}</span></a>
		  {% endfor %}
	  {% endif %}
	</div>
	<div class="post-content light">
	  {{ post.excerpt }}
	  <p><a href="{{ BASE_PATH }}{{ post.url }}">Read more</a></p>
	</div>
</article>
{% endfor %}

<a href="/archive.html">Earlier posts &raquo;</a>
