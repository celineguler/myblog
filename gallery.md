---
layout: page
title: gallery
permalink: /gallery/
---

<style>
/* Remove default padding/margin from page wrapper */
.page, .wrapper, .content {
  max-width: none !important;
  padding: 2 !important;
  margin: 2 !important;
}

/* Gallery layout using CSS Grid */
.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  padding: 20px;
  width: 100vw;
  box-sizing: border-box;
}

.image-card {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.image-card img {
  width: 100%;
  height: auto;
  border-radius: 8px;
  display: block;
  object-fit: cover;
}

.image-card p {
  text-align: center;
  margin-top: 8px;
  font-size: 14px;
  color: #555;
}

/* Responsive grid columns */
@media (max-width: 1200px) {
  .gallery {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 800px) {
  .gallery {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 500px) {
  .gallery {
    grid-template-columns: 1fr;
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
