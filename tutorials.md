---
layout: page
title: Tutorials
permalink: /tutorials/
---

<ul>
{% for tut in site.tutorials %}
  <li><a href="{{site.baseurl}}/tutorials/{{tut.slug}}">{{ tut.name }}</a></li>
{% endfor %}
</ul>
