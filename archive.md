---
layout: base
---

{% assign grouped_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year in grouped_by_year %}
<h1>{{ year.name }}</h1>
  {% assign grouped_by_month = year.items | group_by_exp: "post", "post.date | date: '%B'" %}
  {% for month in grouped_by_month %}
<h2>{{ month.name }}</h2>
<ul>
      {% for post in month.items %}
<li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
      {% endfor %}
</ul>
  {% endfor %}
{% endfor %}