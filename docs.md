---
layout: page
title: Docs
permalink: /docs/
---

Using resources we describe to UD the system piece by piece.

Designing an application, in UD terms, means to create a set of json files, or <a href="{{site.baseurl}}/docs/resource">resources</a>, we provide to the system in order to describe the **interface** and the **behaviour** of the application. Each resource is a json file with a unique name, a unique path and a specific type.

<ul>
{% for doc in site.docs %}
  <li><a href="{{site.baseurl}}/docs/{{doc.slug}}">{{ doc.name }}</a></li>
{% endfor %}
</ul>

### Resources:

<ul>
{% for resource in site.resources %}
  <li><a href="{{site.baseurl}}/resources/{{resource.slug}}">{{ resource.name }}</a></li>
{% endfor %}
</ul>

