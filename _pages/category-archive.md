---
title: "Posts by Category"
layout: archive
permalink: /categories/
author_profile: true
---

{% comment %}based on https://github.com/mushishi78/jekyll-group-by-array{% endcomment %}

{% include group-by-array collection=site.documents field='categories' %}

<div>
    {% for category in group_names %}
    {% assign posts = group_items[forloop.index0] %}

    {% assign category_with_replaced_undescores = category | replace: "_", "-" %}

    {{ output_string }}

        <h2 id="{{ category_with_replaced_undescores  }}">{{ category_with_replaced_undescores  }}</h2>
        <ul>
            {% for post in posts %}
            <li>
                <a href='{{ site.baseurl }}{{ post.url }}'>{{ post.title }}</a>
            </li>
            {% endfor %}
        </ul>
    {% endfor %}
</div>
