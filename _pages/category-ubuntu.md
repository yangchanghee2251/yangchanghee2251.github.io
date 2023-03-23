---
title: "ubuntu"
layout: archive
permalink: /ubuntu
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.ubuntu %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}