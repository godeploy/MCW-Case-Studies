---
title: Case Studies
permalink: index.html
layout: home
---

# go deploy - Azure Case Studies

Hyperlinks to each of the case studies are listed below.

## Case Studies

{% assign labs = site.pages | where_exp: "page", "page.url contains '/docs'" %}
| Case Study |
| --- |
{% for activity in labs  %}{% if activity.lab.title %}| {{ activity.lab.title }} |
{% endif %}{% endfor %}
