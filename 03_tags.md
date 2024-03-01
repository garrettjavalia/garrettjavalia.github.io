---
layout: page
title: "태그"
permalink: /tags/
---

제 Github Page에는 아래와 같은 태그가 있습니다.
{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}

<details>
  <summary>{{ tag[0] }}</summary>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
</details>
{% endfor %}
