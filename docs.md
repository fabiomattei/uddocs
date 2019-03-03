---
layout: page
title: Docs
permalink: /docs/
---

My docs index

{% for doc in site.docs %}
  <h2><a href="{{site.baseurl}}/docs/{{doc.slug}}">{{ doc.name }}</a></h2>

{% endfor %}

ciao

