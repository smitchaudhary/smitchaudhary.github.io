---
layout: single
title: Blog
permalink: /blog/
classes: wide
header:
  overlay_color: "#0000"
  overlay_filter: "0.8"
  overlay_image: /assets/images/blog_header.jpg
---

{% for post in site.posts %}
  {% if post.categories contains "blog" %}
    {% include archive-single.html %}
  {% endif %}
{% endfor %}
