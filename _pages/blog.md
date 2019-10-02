---
layout: archive
title: "Page Archive"
permalink: /blog/
author_profile: true
---

{% for post in site.pages %}
  {% include archive-single.html %}
{% endfor %}
