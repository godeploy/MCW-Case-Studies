---
title: Case Studies
permalink: index.html
layout: home
---

# go deploy - Azure Case Studies

Hyperlinks to each of the case studies are listed below.

{% assign labs = site.pages | where_exp: "page", "page.url contains '/docs'" %}
| Link |
| --- |
[{{ activity.lab.title }}{% if activity.lab.type %} {% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endif %}{% endfor %}
