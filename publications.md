---
layout: page
title: Publications
subtitle: Research papers, preprints, and project pages.
permalink: /publications/
---

{% assign publications = site.publications | sort: "date" | reverse %}
{% assign grouped_publications = publications | group_by_exp: "publication", "publication.year" %}

{% if publications.size > 0 %}
{% for year_group in grouped_publications %}
<h2 class="year-heading">{{ year_group.name }}</h2>
<div class="publication-list">
{% for publication in year_group.items %}
{% include publication-card.html publication=publication %}
{% endfor %}
</div>
{% endfor %}
{% else %}
<p>No publications have been added yet.</p>
{% endif %}
