---
layout: page
title: gallery
permalink: /gallery/
---

<style>
.gallery {
  column-count: 3;
  column-gap: 20px;
  padding: 40px;
}

.image-card {
  break-inside: avoid;
  margin-bottom: 20px;
  display: inline-block;
  width: 100%;
}

.image-card img {
  width: 100%;
  height: auto;
  border-radius: 8px;
  display: block;
}

.image-card p {
  text-align: center;
  margin-top: 8px;
  font-size: 14px;
  color: #555;
}
</style>

<div class="gallery">
  {% for image in site.data.gallery %}
    <div class="image-card">
      <img src="{{ image.url }}" alt="{{ image.alt | default: 'gallery image' }}">
      {% if image.title %}
        <p>{{ image.title }}</p>
      {% endif %}
    </div>
  {% endfor %}
</div>
