---
layout: default
title: Home
---

FPGA engineer writing about high-speed digital design, embedded systems, and the debugging patterns that don't show up in a datasheet.

{% for post in site.posts %}
### [{{ post.title }}]({{ post.url | relative_url }})
*{{ post.date | date: "%B %-d, %Y" }}*

{{ post.excerpt }}

{% endfor %}
