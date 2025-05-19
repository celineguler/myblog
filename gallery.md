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
  gap: 40px;
  padding: 40px;
}

.image-card {
  text-align: center;
  max-width: 300px;
}

.image-card img {
  width: 100%;
  height: auto;
  display: block;
  margin: 0 auto;
  border-radius: 8px;
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
