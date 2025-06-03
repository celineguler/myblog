---
layout: page
title: gallery
permalink: /gallery/
---

<style>
/* Reset body margin/padding to take full width but with symmetric breathing room */
body {
  margin: 0;
  padding: 0;
  width: 100%;
}

/* Masonry-style gallery with symmetric padding */
.gallery {
  column-count: 3;
  column-gap: 17px;
  padding: 20px 40px; /* Top-bottom: 20px, Left-right: 40px */
  box-sizing: border-box;
  width: 100%;
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
    padding: 20px 30px;
  }
}

@media (max-width: 500px) {
  .gallery {
    column-count: 1;
    padding: 20px 20px;
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
