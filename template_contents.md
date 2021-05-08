---
layout: page 
title: 目录 
permalink: /contents/
---

<div class="post-list" itemscope="" itemtype="http://schema.org/Blog">
{% capture labels = "" %}
    {% for postss in site.posts %}
        {% for plabel in postss.labels %}
            {{ labels | append: plabel }}
        {% endfor %}
    {% endfor %}
{% endcapture %}
{% assign list = labels | split: " " %}
{% assign list = list | uniq | join: "," %}
{% assign list = list | split: "," %}
{% for label in list %}
    {% include /contents/labels_card.html %}
{% endfor %}
</div>