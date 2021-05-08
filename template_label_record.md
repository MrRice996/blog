---
layout: page
title: Record
permalink: /labels/Record/
---

<div class="post-list" itemscope="" >
    {% for post in site.posts %}
        {% for labelss in post.labels %}
            {% if labelss contains 'Record' %}
                {% include /card.html %}
            {% endif %}
        {% endfor %}
    {% endfor %}
</div>