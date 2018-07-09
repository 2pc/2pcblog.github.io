---
layout: page
title: 2pc's
tagline: 
---
{% include JB/setup %}

###  Across the Great Wall we can reach every corner in the world 

## Latest Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; {{ post.draft_flag }} <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>




