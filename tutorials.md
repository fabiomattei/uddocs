---
layout: page
title: Tutorials
permalink: /tutorials/
---

My tutorials index

{% for tut in site.tutorials %}
  <h2><a href="{{site.baseurl}}/tutorials/{{tut.slug}}">{{ tut.name }}</a></h2>

{% endfor %}