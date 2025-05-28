---
layout: page
title: gallery
permalink: /gallery/
---

<style>
/* Remove default padding/margin from page wrapper */
.page, .wrapper, .content {
  max-width: none !important;
  padding: 0 !important;
  margin: 0 !important;
}

/* Masonry-style gallery layout */
.gallery {
  column-count: 4;
  column-gap: 20px;
  padding: 20px;
  width: 100vw;
  box-sizing: border-box;
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

/* Responsive columns */
@media (max-width: 1200px) {
  .gallery {
    column-count: 3;
  }
}

@media (max-width: 800px) {
  .gallery {
    column-count: 2;
  }
}

@media (max-width: 500px) {
  .gallery {
    column-count: 1;
  }
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
