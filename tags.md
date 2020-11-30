---
layout: page
title: "Tags"
description: "Posts indexed by tag"
permalink: /tags/
---

<div id="tags">
{% assign sorted_tags = site.tags | sort %}
  <h2>All tags</h2>
    <p>
    {% for tag in sorted_tags %}
      <code class="highlighter-rouge"><a class="post-tag" href="/tags/#{{ tag[0] | slugify }}">{{ tag[0] }}</a></code>
    {% endfor %}
    </p>
  <h2>Posts by tag</h2>
  {% for tag in sorted_tags %}
    <div id="{{ tag[0] | slugify }}">
      <h3>{{ tag[0] }}</h3>
      {% for post in tag[1] %}
        <ul>
          <p>
            <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          </p>
        </ul>
      {% endfor %}
    </div>
  {% endfor %}
</div>
