---
layout: single
title: Readings
excerpt: Paper expositions and tutorials
permalink: /readings/
classes: wide
header:
  overlay_color: "#0000"
  overlay_filter: "0.8"
  overlay_image: /assets/images/readings.jpg
author_profile: true
---

## What is this? And why am I doing this?

As is usual for a student at graduate level, I read a bunch of academic articles, books, etc.
I have decided to write expositions/summaries for some of them where the level of detail would vary.
The motivation behind this is three-fold.
Firstly, writing it down, and having to explain something greatly enhances my level of understanding.
Moreover, for the material that I do not directly use, the details get lost on me after a while.
Thus, it is a way to expand the breadth as well as depth.
Additionally, I want to improve my (technical) writing skills, and what better way than *just do it*?

{% for post in site.posts %}
  {% if post.categories contains "readings" %}
    {% include archive-single.html %}
  {% endif %}
{% endfor %}
