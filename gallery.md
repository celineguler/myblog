---
layout: page
title: gallery
permalink: /gallery/
---

<style>
/* Remove strict max-width, but keep a soft margin */
.page, .wrapper, .content {
  max-width: 100% !important;
  margin: 0 auto;
  padding: 0;
}

/* Add side margins to gallery */
.gallery-container {
  padding: 0 40px;
}

/* Grid layout: 3 per row */
.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 30px;
  padding: 40px 0;
}

/* Each image card */
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
}

.image-card p {
  margin-top: 8px;
  font-size: 14px;
  color: #555;
  text-align: center;
}

/* Responsive layout */
@media (max-width: 1000px) {
  .gallery {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 600px) {
  .gallery {
    grid-template-columns: repeat(1, 1fr);
  }
}
</style>

<div class="gallery-container">
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
</div>
