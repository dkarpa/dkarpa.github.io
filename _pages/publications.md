---
layout: page
permalink: /publications/
title: publications
description: Publications.
nav: true
nav_order: 2
---

<!-- _pages/publications.md -->

<!-- Bibsearch Feature -->

{% include bib_search.liquid %}

<div class="publications">

<h2 class="bibliography">Books and book chapters</h2>

{% bibliography --group_by none --query @*[category=book] %}

<h2 class="bibliography">Journal articles</h2>

{% bibliography --group_by none --query @*[category=journal] %}

<h2 class="bibliography">Working papers</h2>

{% bibliography --group_by none --query @*[category=working_paper] %}

</div>
