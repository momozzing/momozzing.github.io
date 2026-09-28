---
title: "📄 논문리뷰"
permalink: /categories/Paper_review/
layout: archive
author_profile: true
---

챗봇·NLP·LLM·Agent 관련 논문을 읽고 정리하는 공간입니다. 분야를 누르면 그 분야 논문이 순서대로 나옵니다.

<div class="paper-fields">
{% for f in site.data.paper_fields %}
  {% assign fposts = site.posts | where: "field", f.key | sort: "date" %}
  {% assign n = 0 %}{% for p in fposts %}{% if p.categories contains 'Paper review' %}{% assign n = n | plus: 1 %}{% endif %}{% endfor %}
  <a class="paper-fields__card" href="{{ '/paper-review/' | append: f.key | append: '/' | relative_url }}">
    <span class="paper-fields__title">{{ f.title }}</span>
    <span class="paper-fields__meta">{{ n }}편 · {{ fposts.first.date | date: "%Y" }}{% assign y2 = fposts.last.date | date: "%Y" %}{% assign y1 = fposts.first.date | date: "%Y" %}{% if y1 != y2 %}~{{ y2 }}{% endif %}</span>
    <span class="paper-fields__desc">{{ f.desc }}</span>
  </a>
{% endfor %}
</div>
