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
<div class="project-grid">
{% for project in projects %}
  <a class="project-card" href="{{ project.url }}">

    {% if project.thumb and project.thumb != "" %}
      <div class="project-card__media"
           style="background-image: url('{{ site.urlimg }}{{ project.thumb }}');"></div>
    {% else %}
      <div class="project-card__media project-card__media--placeholder"
           {% if project.placeholder_color %}style="background-color: {{ project.placeholder_color }};"{% endif %}>
        <span class="project-card__placeholder-text">
          {{ project.placeholder_label | default: project.title }}
        </span>
      </div>
    {% endif %}

    <div class="project-card__body">
      <h4 class="project-card__title">{{ project.title }}</h4>

      {% if project.organization or project.date_range %}
        <p class="project-card__meta">
          {{ project.organization }}{% if project.organization and project.date_range %} · {% endif %}{{ project.date_range }}
        </p>
      {% endif %}

      {% if project.tagline %}
        <p class="project-card__tagline">{{ project.tagline }}</p>
      {% endif %}

      {% if project.tech %}
        <p class="project-card__tags">
          {% for t in project.tech limit:4 %}<span class="project-card__tag">{{ t }}</span>{% endfor %}
        </p>
      {% endif %}
    </div>
  </a>
{% endfor %}
</div>