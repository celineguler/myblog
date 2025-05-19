---
layout: page
title: gallery
permalink: /gallery/
---

<div class="gallery">
  {% for image in site.data.gallery %}
    <div class="image-card">
      <img src="{{ image.url }}" alt="{{ image.alt }}">
      <p>{{ image.title }}</p>
    </div>
  {% endfor %}
</div>
