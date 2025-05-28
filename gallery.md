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
  gap: 30px;
  padding: 40px;
}

.image-card {
  flex: 0 1 calc(30% - 20px); /* 3 per row with spacing */
  box-sizing: border-box;
  text-align: center;
}

.image-card img {
  width: 100%;
  height: auto;
  border-radius: 10px;
  display: block;
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
