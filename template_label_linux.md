---
layout: page
title: Linux
permalink: /labels/Linux/
---

<div class="post-list" itemscope="" >
    {% for post in site.posts %}
        {% for labelss in post.labels %}
            {% if labelss contains 'Linux' %}
                {% include /card.html %}
            {% endif %}
        {% endfor %}
    {% endfor %}
</div>