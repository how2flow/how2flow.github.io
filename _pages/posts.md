---
permalink: /posts/
title: Posts
comments: false
entries_layout: grid
layout: single
---

<div class="home-intro" style="margin-bottom: 2em; padding: 1.5em; background: #fdfdfd;">
  <h2 style="margin-top: 0; color: #222;">🌊 How Should Data Flow?</h2>
  <p style="font-size: 1.1em; line-height: 1.6; color: #444;">
    Welcome to the <strong>How To Flow</strong> technical archive. <br>
  </p>
  <p style="font-size: 1.0em; line-height: 1.6; color: #666;">
    This is my personal space to document the <strong>flow of technology</strong>, focusing on Embedded Linux, System Architecture, and Device Drivers.<br>
  </p>
</div>

<h3 class="archive__subtitle">{{ "Lists" }}</h3>

{% assign posts = site.posts %}


{% assign entries_layout = page.entries_layout | default: 'list' %}
<div class="entries-{{ entries_layout }}">
  {% for post in posts %}
    {% include archive-single.html type=entries_layout %}
  {% endfor %}
</div>
