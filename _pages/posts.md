---
title: "Blog"
permalink: /posts/
layout: archive
author_profile: true
---

Technical notes on RL, robot dynamics, simulation, and deployment.

{% assign posts = site.posts %}
{% assign entries_layout = page.entries_layout | default: 'list' %}
<div class="entries-{{ entries_layout }}">
  {% for post in posts %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
</div>
