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
    <a href="{{ post.url | relative_url }}" class="post-tile">

      <div class="tile-images">
        {% for img in post.images_data limit:3 %}
        <div class="tile-image"
             style="background-image: url('{{ post.image_folder }}{{ img.file | relative_url }}');">
        </div>
        {% endfor %}
      </div>

      <div class="tile-overlay">
        <h2>{{ post.title }}</h2>
        <span>{{ post.date | date: "%B %Y" }}</span>
      </div>

    </a>
  {% endfor %}
</section>
