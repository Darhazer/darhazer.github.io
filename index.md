---
layout: page
title: Latests posts
tagline: Development and testing
---
{% include JB/setup %}


  {% for post in site.posts %} 
##[{{ post.title }}]({{ BASE_PATH }}{{ post.url }}) ( {{post.date | date_to_string}} )
  {% endfor %}

