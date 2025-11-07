---
layout: default
title: Posts
permalink: /posts/
---

## {{ page.title }}

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> {{ post.date }}
    </li>
  {% endfor %}
</ul>

