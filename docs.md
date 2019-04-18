---
layout: page
title: Docs
permalink: /docs/
---

Using resources we describe the system piece by piece to UD.

Designing a system, in UD terms, means to create a set of json file, or resources, we provide to the system in order to describe the interface and the behaviour. Each resource is a json file with a unique name, a unique path and a specific type.

<ul>
{% for doc in site.docs %}
  <li><a href="{{site.baseurl}}/docs/{{doc.slug}}">{{ doc.name }}</a></li>
{% endfor %}
</ul>
