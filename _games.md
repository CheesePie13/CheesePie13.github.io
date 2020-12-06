---
#
layout: page
title: Games
permalink: /games/
---

<ul>
	{% for post in site.posts %}
		{% if post.categories contains "games" %}
			<li>
				<a href="{{ post.url }}">{{ post.title }}</a>
			</li>
		{% endif %}
	{% endfor %}
</ul>
