---
layout: page
title: gallery
permalink: /gallery/
---

<style>
/* Remove global padding but keep a little side margin */
.page, .wrapper, .content {
  max-width: none !important;
  padding: 0 !important;
  margin: 0 auto !important;
}

.gallery {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 30px;
  padding: 40px 60px; /* top-bottom: 40px, left-right: 60px */
  box-sizing: border-box;
}

.image-card {
  flex: 0 1 calc(33.333% - 20px); /* 3 per row with gap adjustment */
  box-sizing: border-box;
  text-align: center;
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
}

/* Responsive tweaks */
@media (max-width: 1000px) {
  .image-card {
    flex: 0 1 calc(50% - 20px); /* 2 per row */
  }
}

@media (max-width: 600px) {
  .image-card {
    flex: 0 1 100%; /* 1 per row */
  }
}
</style>

<div class="gallery">
  {% for image in site.data.gallery %}
