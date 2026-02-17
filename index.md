---
layout: default
---

<section class="homepage-hero">
  <h1>Photography</h1>
  <p>A study of light, silence, and presence.</p>
</section>

<section class="post-grid">

  {% for post in site.posts %}
    {% assign folder = post.image_folder %}
    {% assign images = "" | split: "" %}

    {% for file in site.static_files %}
      {% if file.path contains folder %}
        {% assign images = images | push: file.path %}
      {% endif %}
    {% endfor %}

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

  {% endfor %}

</section>
