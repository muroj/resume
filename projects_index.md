---
title: Projects
layout: default
permalink: /projects/
---

# Projects

Below are project writeups. Click a project to view details.

<ul>
{% for p in site.projects %}
  <li><a href="{{ p.url | relative_url }}">{{ p.title }}</a></li>
{% endfor %}
</ul>
