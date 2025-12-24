---
permalink: /legacy/
title: Posts
comments: false
entries_layout: grid
layout: single
---

It is a space where you record and post simple development projects<br>
or things you need while developing.<br>
<br>
You can also find various posts through search.<br>

<h3 class="archive__subtitle">{{ "Legacy Lists" }}</h3>

{% assign legacy = site.legacy %}

{% assign entries_layout = page.entries_layout | default: 'list' %}
<div class="entries-{{ entries_layout }}">
  {% for post in legacy %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
</div>
