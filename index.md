---
title: Case Studies
permalink: index.html
layout: home
---

# go deploy - Azure Case Studies

Hyperlinks to each of the case studies are listed below.

## Case Studies

{% assign labs = site.pages | where_exp: "page", "page.url contains '/docs'" %}
| Module | Lab |
| --- | --- |
{% for activity in labs  %}{% if activity.lab.az204Module %}| {{ activity.lab.az204Module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endif %}{% endfor %}
