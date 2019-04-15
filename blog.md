---
layout: default
title: Blog
permalink: /index.html
---

{% for post in site.posts %}

<div class="post-summary">
  
### [{{ post.title }}]({{ post.url | prepend: site.baseurl }})

{{ post.excerpt }}

_{{ post.date | date: "%b %-d, %Y" }}_ 

</div>
{% endfor %}