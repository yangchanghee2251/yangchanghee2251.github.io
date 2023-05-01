---
title: "Diffusion"
layout: archive
permalink: /diffusion
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.diffusion %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}