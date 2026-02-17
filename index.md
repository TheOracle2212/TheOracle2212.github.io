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

    {% assign folder = post.image_folder | replace: "/", "" %}
    {% assign images = "" | split: "" %}

    {% if folder %}

      {% for file in site.static_files %}
        {% assign filepath = file.path | replace: "/", "" %}
        
        {% if filepath contains folder %}
          {% if file.extname == ".jpg" 
             or file.extname == ".jpeg" 
             or file.extname == ".png" 
             or file.extname == ".webp" %}
             
            {% assign images = images | push: file.path %}
            
          {% endif %}
        {% endif %}
      {% endfor %}

      {% assign images = images | sort %}

      {% if images.size > 0 %}

        <a href="{{ post.url | relative_url }}" class="post-tile">

          <div class="tile-images">
            {% for img in images limit:3 %}
              <div class="tile-image"
                   style="background-image: url('{{ img | relative_url }}');">
              </div>
            {% endfor %}
          </div>

          <div class="tile-overlay">
            <h2>{{ post.title }}</h2>
            <span>{{ post.date | date: "%B %Y" }}</span>
          </div>

        </a>

      {% endif %}

    {% endif %}

  {% endfor %}

</section>
