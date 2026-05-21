---
layout: page-fullwidth
title: "Projects"
subheadline: "Embedded, FPGA, and hardware design"
teaser: "A selection of things I've built — at school, at work, and at home."
permalink: "/portfolio/"
header:
    image_fullwidth: "header_drop.jpg"
---

{% assign projects = site.projects | sort: "order" %}
<div class="row">
{% for project in projects %}
  <div class="medium-6 large-4 columns b30">
    <a href="{{ project.url }}">
      <img src="{{ site.urlimg }}{{ project.thumb }}" alt="{{ project.title }}">
    </a>
    <h4><a href="{{ project.url }}">{{ project.title }}</a></h4>
    <p class="teaser">{{ project.tagline }}</p>
    {% if project.tech %}
      <p>{% for t in project.tech limit:4 %}<span class="label">{{ t }}</span> {% endfor %}</p>
    {% endif %}
  </div>
{% endfor %}
</div>