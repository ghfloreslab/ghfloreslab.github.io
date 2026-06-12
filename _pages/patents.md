---
layout: page
permalink: /patents/
title: Patents
description: Inventor and co-inventor of 7 patents
years: [2022, 2021, 2020]
nav: true
nav_order: 2
---
<!-- _pages/patests.md -->
<div class="publications">

{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f patents -q @*[year={{y}}]* %}
{% endfor %}

</div>
