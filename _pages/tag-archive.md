---
title: "Posts by Tag"
permalink: /tags/
layout: archive
author_profile: true
---

{% comment %}based on https://github.com/mushishi78/jekyll-group-by-array{% endcomment %}

{% include group-by-array collection=site.documents field='tags' %}

<ul>
    {% for tag in group_names %}
    {% assign posts = group_items[forloop.index0] %}

    <li>
        <h2>{{ tag }}</h2>
        <ul>
            {% for post in posts %}
            <li>
                <a href='{{ site.baseurl }}{{ post.url }}'>{{ post.title }}</a>
            </li>
            {% endfor %}
        </ul>
    </li>
    {% endfor %}
</ul>
