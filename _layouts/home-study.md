---
layout: archive
---

{{ content }}

<h3 class="archive__subtitle">{{ site.data.ui-text[site.locale].study_lists | default: "Study Lists" }}</h3>

{% if paginator %}
  {% assign posts = paginator.study %}
{% else %}
  {% assign posts = site.study %}
{% endif %}

{% assign entries_layout = page.entries_layout | default: 'list' %}
<div class="entries-{{ entries_layout }}">
  {% for post in posts limit:8 %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
</div>

{% include paginator.html %}
