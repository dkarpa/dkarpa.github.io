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

<h2 class="bibliography">Selected work in progress</h2>

<ul>
  <li>“Sorry, None of Our Business: Economic Grievance, Zero-Sum Thinking, and Support for Defending Ukraine” (with Lasha Chargaziia and Michael Rochlitz)</li>
  <li>“Deprivation, Public Discontent and Support for Authoritarian Policies” (with Michael Rochlitz) [<a href="/assets/pdf/Karpa_Rochlitz_Deprivation_slides.pdf">slides</a>]</li>
  <li>“‘I Am Not an Activist’: The Digital Authoritarian Bargain in Kazakhstan” (with Michael Rochlitz) [<a href="/assets/pdf/Karpa_Rochlitz_Not_an_Activist_slides.pdf">slides</a>]</li>
  <li>“Public Opinion in Wartime Russia: Internet Restrictions, Repression, and Support for the War” (with Andrey Tkachenko, Lasha Chargaziia and Timothy M. Frye)</li>
  <li>“Legitimacy of Electronic Travel Authorisation in the United Kingdom” (with Daria Gritsenko)</li>
  <li>“How Ends Motivate Means: Unpacking How Motivated Reasoning Shapes Legitimacy Judgments of Automated Travel Authorisation Systems” (with Daria Gritsenko)</li>
</ul>

</div>
