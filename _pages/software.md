---
layout: page
permalink: /software/
title: software
description: Open-source software I develop and maintain for political-science research.
nav: true
nav_order: 3
display_categories: [software]
horizontal: false
---

<!-- _pages/software.md -->
<div class="projects">
{% assign software_projects = site.projects | where: "category", "software" | sort: "importance" %}
{% if page.horizontal %}
<div class="container">
  <div class="row row-cols-1 row-cols-md-2">
  {% for project in software_projects %}
    {% include projects_horizontal.liquid %}
  {% endfor %}
  </div>
</div>
{% else %}
<div class="row row-cols-1 row-cols-md-3">
  {% for project in software_projects %}
    {% include projects.liquid %}
  {% endfor %}
</div>
{% endif %}
</div>
