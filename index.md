---
layout: default
title: "Craig Russell Technical Blog"
---
# Hello World

Rumblings of a new blog post.


<h1>Latest Posts</h1>
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">
        <b>{{post.title}}</b> ({{post.date | date: "%b %d, %Y" }})
        <p>
        {% if post.description %}
           <i>{{ post.description }}</i>
        {% endif %}       
        </p>
      </a>
    </li>
  {% endfor %}
</ul>
