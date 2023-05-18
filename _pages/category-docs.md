---
title: "Document"
layout: archive
permalink: categories/docs
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories['Docs'] %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}