---
layout: page
permalink: /publications/
title: publications
sections:
  - bibquery: "@article"
    text: "Conference and Journal articles"
  - bibquery: "@inproceedings"
    text: "Workshop papers"
  - bibquery: "@misc|@phdthesis|@mastersthesis"
    text: "Thesis"
years: [2024, 2023, 2022, 2018]
nav: true
nav_order: 1
---
<!-- _pages/publications.md -->
<div class="publications">

{%- for section in page.sections %}
  <a id="{{section.text}}"></a>
  <p class="bibtitle">{{section.text}}</p>
  {%- for y in page.years %}

    {%- capture citecount -%}
        {%- bibliography_count -f {{site.scholar.bibliography}} -q {{section.bibquery}}[year={{y}}] -%}
    {%- endcapture -%}

    {%- if citecount != "0" %}
      {% bibliography -f {{site.scholar.bibliography}} -q {{section.bibquery}}[year={{y}}] %}
    {%- endif -%}

  {%- endfor %}
{%- endfor %}


</div>
