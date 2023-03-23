---
title: "blog"
layout: archive
permalink: /jekyll
---


{% assign posts = site.categories.jekyll %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}