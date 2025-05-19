---
layout: page
title: gallery
permalink: /gallery/
---

<style>
.gallery {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 170px;
  padding: 40px;
}


</style>

<div class="gallery">
  {% for image in site.data.gallery %}
    <div class="image-card">
      <img src="{{ image.url }}" alt="{{ image.alt }}">
      <p>{{ image.title }}</p>
    </div>
  {% endfor %}
</div>
