---
layout: page
title: char *welcome; (home)
tagline: Supporting tagline
---
{% include JB/setup %}

This site was born to supplement the project I'm building: [Ruby Design Patterns](http://alistanis.github.io/DesignPatterns/).

In time I hope it to become a valuable resource for Design Patterns, Ruby, and C/Ruby C Extensions.

Complete code for the project available at: [Github](https://github.com/Alistanis/DesignPatterns)

## Tutorials

[Check out the categories page for an up to date list of tutorials by category.](/categories.html)

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>