---
layout: single
title: Projects
excerpt: Fun and academic projects
permalink: /projects/
classes: wide
header:
  overlay_color: "#0000"
  overlay_filter: "0.8"
  overlay_image: /assets/images/projects_header.png
author_profile: true
---

{% for post in site.posts %}
  {% if post.categories contains "project" %}
    {% include archive-single.html %}
  {% endif %}
{% endfor %}
