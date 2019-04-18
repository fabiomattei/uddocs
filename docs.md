---
layout: page
title: Docs
permalink: /docs/
---

<ul>
{% for doc in site.docs %}
  <li><a href="{{site.baseurl}}/docs/{{doc.slug}}">{{ doc.name }}</a></li>
{% endfor %}
</ul>
