---
layout: page
permalink: /gallery/
---

<style>
/* Reset layout margins/paddings, but preserve inner spacing */
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
}

/* Maintain header and title padding as intended */
.page {
  max-width: 100%;
  padding: 40px; /* Adjust if needed for header/title spacing */
  box-sizing: border-box;
}

/* Optional: Ensure .wrapper and .content don’t override layout */
.wrapper, .content {
  max-width: 100%;
  box-sizing: border-box;
}

/* Gallery layout */
.gallery {
  column-count: 3;
  column-gap: 17px;
  padding: 20px 40px;
  width: 100%;
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

/* Responsive gallery column counts */
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

<!-- The gallery title is handled by the layout: page (likely in your theme's layout file) -->

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
