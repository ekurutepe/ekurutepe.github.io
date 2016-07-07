---
layout: default
title: Blog
permalink: /blog/
---

{% for post in site.posts %}

<div class="post_summary">
  
### [{{ post.title }}]({{ post.url | prepend: site.baseurl }})

{{ post.excerpt }}

_{{ post.date | date: "%b %-d, %Y" }}_ 

</div>
{% endfor %}