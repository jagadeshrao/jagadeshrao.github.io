---
layout: default
title: Blog
---

<p class="intro">Notes on FPGA design, high-speed digital interfaces, and the debugging patterns that don''t show up in a datasheet.</p>

<div class="post-list">
{% for post in site.posts %}
  <article class="post-preview">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p class="post-date">{{ post.date | date: "%B %-d, %Y" }}</p>
    <p class="post-excerpt">{{ post.excerpt | strip_html }}</p>
  </article>
{% endfor %}
</div>
