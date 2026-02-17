---
layout: default
title: Home
---

<section class="homepage-hero">
  <h1>Photography</h1>
  <p>A visual journal of light, motion, and presence.</p>
</section>

<section class="post-grid">
  {% for post in site.posts %}
    {% assign first_image = post.images_data[0] %}
    <a href="{{ post.url | relative_url }}" class="post-tile">
      
      <div class="tile-image"
           style="background-image: url('{{ post.image_folder }}{{ first_image.file | relative_url }}');">
      </div>

      <div class="tile-overlay">
        <h2>{{ post.title }}</h2>
        <span>{{ post.date | date: "%B %Y" }}</span>
      </div>

    </a>
  {% endfor %}
</section>
