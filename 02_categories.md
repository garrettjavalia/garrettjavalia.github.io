---
layout: page
title: "분류"
permalink: /categories/
---

제 Github Page에는 아래와 같은 분류가 있습니다.
{% assign sorted = site.categories | sort %}
{% for category in sorted %}

  <h3>{{ category[0] }}</h3>
  <ul>
    {% for post in category[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
