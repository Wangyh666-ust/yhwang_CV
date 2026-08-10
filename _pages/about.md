---
layout: about
title: HOMEPAGE
permalink: /
subtitle: Third-year Data Science undergraduate at <a href='https://hkust.edu.hk/'>HKUST</a>

profile:
  align: right
  image: prof_pic.jpg # replace assets/img/prof_pic.jpg to change the photo
  image_circular: false # crops the image to make it circular
  more_info: >
    <p>Hong Kong SAR</p>
    <p>ywangrg@connect.ust.hk</p>

selected_papers: false # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: false # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: false
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

## BACKGROUND

I am a third-year Data Science student at the Hong Kong University of Science and Technology (HKUST), with a strong foundation in machine learning, computer vision, statistical modeling, and programming.

This summer I worked as a Computer Vision intern at Zhijie Exploration Technology (Shenzhen), where I built a metric-scale multi-view 3D reconstruction pipeline for tabletop food volume estimation. Previously, I trained a U-Net model for heart detection and segmentation in X-ray images at Seoul National University (over 90% accuracy, 0.7992 F1-score), and I have conducted research with Prof. Gary Chan at HKUST on making 3D reconstruction more efficient by using 2D floorplans as structural priors. My final-year project explores a personal embodied AI agent driven by reinforcement learning.

## SKILLS

- **Programming:** Python, C++, Java, R
- **Machine Learning & Data:** PyTorch, NumPy, pandas
- **Visualization:** Tableau, Datawrapper, iNZight, RAWGraphs
- **Languages:** Mandarin (native), English

## PROJECTS

A quick overview — click any title for the dedicated detail page, or browse the full [projects]({{ '/projects/' | relative_url }}) page.

{% assign categories = "research,projects" | split: "," %}
{% for category in categories %}

### {{ category | capitalize }}

{% assign projs = site.projects | where: "category", category | sort: "importance" %}
{% for p in projs %}

- **[{{ p.title }}]({{ p.url | relative_url }})** — {{ p.description }}
{% endfor %}
{% endfor %}

## INTERESTS & DIRECTIONS

I am seeking opportunities in **data analysis** and **AI/CV research**, for both graduate study and industry roles. My current interests include computer vision and 3D reconstruction, embodied AI, and applied machine learning on real-world problems. The [CV]({{ '/cv/' | relative_url }}) page has my full background.
