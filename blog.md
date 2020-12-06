---
layout: page
title: Blog
permalink: /blog/
---

<ul class="post-list">
	{% assign blog_posts = site.posts | where_exp:"post", "post.categories contains 'blog'" %}
	{% for post in blog_posts %}
		{% include post-list-item.html post=post %}
	{% endfor %}
</ul>
