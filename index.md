---
layout: default
title: Home
description: Academic homepage
profile_image:
social_links:
  - label: GitHub
    url: https://github.com/peer-rh
  - label: Google Scholar
    url:
  - label: LinkedIn
    url:
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
    I am a researcher working on topics at the intersection of machine learning,
    systems, and human-centered technology. Replace this paragraph with a concise
    research biography, your current role, advisors or collaborators, and the
    problems you care about.
  </p>

  <h2 class="section-title">News</h2>
  <ul class="news-list">
    <li><strong>Jun 2026</strong> This site is now generated from Markdown with Jekyll.</li>
    <li><strong>May 2026</strong> Add recent papers, talks, or professional updates here.</li>
  </ul>
</div>
