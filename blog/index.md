---
layout: default
title: Blog posts
---

## Blog posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{post.date | date: '%B %d, %Y'}} - {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
