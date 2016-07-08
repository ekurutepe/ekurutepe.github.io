---
layout: default
title: Projects
permalink: /projects
---

Here is a list of selected projects I've been lucky enough to be involved with in the recent years:

{% for project in site.data.projects %}

<div class="project-container">

### [{{project.title}}]({{project.url}})
_{{project.responsibilities}}_
<div class="li-img">
{% if project.image %}
<img src="{{ project.image }}" class="project-thumb">
{% else %}
<img src="http://placehold.it/160x160" class="project-thumb">
{% endif %} 
</div>
<div class="li-text">
{{project.description}}
</div>
</div>

{% endfor %}

