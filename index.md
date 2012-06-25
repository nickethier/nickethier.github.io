---
layout: page
tagline: Supporting tagline
---
{% for post in site.posts %}
  <header style="margin-bottom: 0em; margin-left:-1em; margin-right: -1em;"><h3 style="margin-bottom: 0px; margin-left: 1em;"><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h3></header>
  <h6 style="padding-left: 10px">&raquo; {{ post.date | date_to_string }}</h6>
  <span>{{post.content}}</span>
{% endfor %}


