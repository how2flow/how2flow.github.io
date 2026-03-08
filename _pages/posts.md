---
permalink: /posts/
title: Posts
comments: false
entries_layout: grid
layout: single
---

<div class="home-intro" style="margin-bottom: 2em; padding: 1.5em; background: #fdfdfd;">
  <h2 style="margin-top: 0; color: #222;">🌊 How Should It Flow?</h2>
  <p style="font-size: 1.1em; line-height: 1.6; color: #444;">
    안녕하세요, <strong>How To Flow</strong> post 공간에 오신 것을 환영합니다. <br>
  </p>
  <p style="font-size: 1.0em; line-height: 1.6; color: #666;">
    이곳은 <strong>기술의 흐름</strong>과 <strong>경제의 흐름</strong>을 끄적이는 개인 공간입니다.<br>
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
