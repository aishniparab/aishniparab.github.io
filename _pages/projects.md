---
layout: page
title: Field Notes on Search
permalink: /search_atlas/
description: A living notebook on intelligence, abstraction, and the geometry of discovery.
nav: true
nav_order: 3
display_categories: [themes]
horizontal: false
---

This notebook begins from a simple question:
How do intelligent systems explore, represent, and abstract vast spaces?

It treats search as a unifying perspective for studying intelligence. The focus is not on search as a single algorithmic process, but as a recurring pattern that links learning, reasoning, and representation. The goal is to understand how structured representations enable efficient exploration, and how principles of search shape both artificial and natural intelligence.

The notes are organized around three areas of inquiry:
1. Formal — mathematics of representation, inference, and optimization.
2. Cognitive — mechanisms of exploration and generalization under constraints.
3. Reflective — processes of innovation and discovery, and how solutions to unknown problems can be searched and made intelligible.

In my current research, I approach these questions through the lens of neurosymbolic modeling, an attempt to bridge the strengths of continuous learning and symbolic reasoning. The challenge lies in combining efficient optimization with structured, interpretable representations. This notebook aims to investigate the three dimensions above through a neurosymbolic lens.

Over time, I hope this notebook will grow into a shared reference—a place to collect ideas, questions, and directions for future work. It is meant to remain unfinished: open to revision, contribution, and reinterpretation as my study of search continues to evolve.

<!-- pages/projects.md -->
<div class="projects">
{% if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {% for category in page.display_categories %}
  <a id="{{ category }}" href=".#{{ category }}">
    <h2 class="category">{{ category }}</h2>
  </a>
  {% assign categorized_projects = site.projects | where: "category", category %}
  {% assign sorted_projects = categorized_projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal %}
  <div class="container">
    <div class="row row-cols-1 row-cols-md-2">
    {% for project in sorted_projects %}
      {% include projects_horizontal.liquid %}
    {% endfor %}
    </div>
  </div>
  {% else %}
  <div class="row row-cols-1 row-cols-md-3">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
  {% endif %}
  {% endfor %}

{% else %}

<!-- Display projects without categories -->

{% assign sorted_projects = site.projects | sort: "importance" %}

  <!-- Generate cards for each project -->

{% if page.horizontal %}

  <div class="container">
    <div class="row row-cols-1 row-cols-md-2">
    {% for project in sorted_projects %}
      {% include projects_horizontal.liquid %}
    {% endfor %}
    </div>
  </div>
  {% else %}
  <div class="row row-cols-1 row-cols-md-3">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
  {% endif %}
{% endif %}
</div>
