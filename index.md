---
layout: default
title: Home
description: Personal academic homepage of Peer Rheinboldt.
profile_image: /assets/profile/peer-rheinboldt.jpg
social_links:
  - label: GitHub
    url: https://github.com/peer-rh
  - label: Google Scholar
    url: https://scholar.google.com/citations?user=YmxzhfYAAAAJ&hl=en
  - label: LinkedIn
    url: https://www.linkedin.com/in/peer-rheinboldt/
  - label: Email
    url: mailto:peer.rheinboldt@gmail.com
---

<section class="home-hero">
  <div>
    <p class="intro-kicker">Hi, I'm</p>
    <h1>{{ site.author.name }}</h1>
    <p class="role">{{ site.author.role }}{% if site.author.affiliation %}, {{ site.author.affiliation }}{% endif %}</p>

    <div class="social-links" aria-label="Social links">
      {% for link in page.social_links %}
        {% if link.url %}
          <a href="{{ link.url }}">{{ link.label }}</a>
        {% endif %}
      {% endfor %}
    </div>
  </div>

  {% if page.profile_image %}
    <img class="profile-photo" src="{{ page.profile_image | relative_url }}" alt="Profile photo">
  {% else %}
    <div class="profile-photo-placeholder" aria-hidden="true">{{ site.author.name | slice: 0 }}</div>
  {% endif %}
</section>

<div class="prose">
  <p>
    I am an MSc Data Science student at ETH Zürich interested in machine learning and efficient language model inference. My current research focuses on speculative decoding, and I enjoy exploring a broad range of topics across AI and computer science.

  </p>

  <h2 class="section-title">News</h2>
  <ul class="news-list">
    <li><strong>Jun 2026</strong> TreeFlash is available as an arXiv preprint.</li>
    <li><strong>Mar 2026</strong> Steering Pretrained Drafters During Speculative Decoding appears at AAAI-26.</li>
  </ul>
</div>
